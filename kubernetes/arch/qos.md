# kubernetes qos

## 资源分类
QOS是kubernetes保证服务质量的一个模块。
先介绍两个基本概览
* 可压缩资源：CPU
在压缩资源部分已经提到CPU属于可压缩资源，当pod使用超过设置的limits值，pod中进程使用cpu会被限制，但不会被kill。

* 不可压缩资源：内存
Kubernetes通过cgroup给pod设置QoS级别，当资源不足时先kill优先级低的pod，在实际使用过程中，通过OOM分数值来实现，OOM分数值从0-1000。
当然磁盘也属于不可压缩的资源

还需要介绍的是k8s设定的三个等级

## 资源评级
### Guaranteed
Every Container in the Pod must have a memory limit and a memory request, and they must be the same.
Every Container in the Pod must have a cpu limit and a cpu request, and they must be the same.
简单说就是pod里面每个容器都必须设定request和limit，并且值必须相同。
eg:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo
spec:
  containers:
  - name: qos-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "700m"
      requests:
        memory: "200Mi"
        cpu: "700m"
```

### Burstable
The Pod does not meet the criteria for QoS class Guaranteed.
At least one Container in the Pod has a memory or cpu request.
至少有一个容器的request或者limit的值设定了，上面的官方的英文说法是有问题的！
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-2
spec:
  containers:
  - name: qos-demo-2-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
```

### BestEffort
the Containers in the Pod must not have any memory or cpu limits or requests.
eg:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-3
spec:
  containers:
  - name: qos-demo-3-ctr
    image: nginx
```

## 源码实现
先看等级的实现pkg/api/v1/helper/qos/qos.go
```go
func GetPodQOS(pod *v1.Pod) v1.PodQOSClass {
	requests := v1.ResourceList{}
	limits := v1.ResourceList{}
	zeroQuantity := resource.MustParse("0")
	isGuaranteed := true
	for _, container := range pod.Spec.Containers {
		// process requests
		for name, quantity := range container.Resources.Requests {
			if !supportedQoSComputeResources.Has(string(name)) {
				continue
			}
			if quantity.Cmp(zeroQuantity) == 1 {
				delta := quantity.Copy()
				if _, exists := requests[name]; !exists {
					requests[name] = *delta
				} else {
					delta.Add(requests[name])
					requests[name] = *delta
				}
			}
		}
		// process limits
		qosLimitsFound := sets.NewString()
		for name, quantity := range container.Resources.Limits {
			if !supportedQoSComputeResources.Has(string(name)) {
				continue
			}
			if quantity.Cmp(zeroQuantity) == 1 {
				qosLimitsFound.Insert(string(name))
				delta := quantity.Copy()
				if _, exists := limits[name]; !exists {
					limits[name] = *delta
				} else {
					delta.Add(limits[name])
					limits[name] = *delta
				}
			}
		}

		if len(qosLimitsFound) != len(supportedQoSComputeResources) {
			isGuaranteed = false
		}
	}
	if len(requests) == 0 && len(limits) == 0 {
		return v1.PodQOSBestEffort
	}
	// Check is requests match limits for all resources.
	if isGuaranteed {
		for name, req := range requests {
			if lim, exists := limits[name]; !exists || lim.Cmp(req) != 0 {
				isGuaranteed = false
				break
			}
		}
	}
	if isGuaranteed &&
		len(requests) == len(limits) {
		return v1.PodQOSGuaranteed
	}
	return v1.PodQOSBurstable
}
```
上面的判定方法就是上面文字叙述的，拿到分级以后就可以对oom事件进行打分了。
看看kubelet里面代码实现kubelet/qos/policy.go
```go

	PodInfraOOMAdj        int = -998
	KubeletOOMScoreAdj    int = -999
	DockerOOMScoreAdj     int = -999
	KubeProxyOOMScoreAdj  int = -999
	guaranteedOOMScoreAdj int = -998
	besteffortOOMScoreAdj int = 1000
	
	
func GetContainerOOMScoreAdjust(pod *v1.Pod, container *v1.Container, memoryCapacity int64) int {
	switch v1qos.GetPodQOS(pod) {
	case v1.PodQOSGuaranteed:
		// Guaranteed containers should be the last to get killed.
		return guaranteedOOMScoreAdj
	case v1.PodQOSBestEffort:
		return besteffortOOMScoreAdj
	}

	// Burstable containers are a middle tier, between Guaranteed and Best-Effort. Ideally,
	// we want to protect Burstable containers that consume less memory than requested.
	// The formula below is a heuristic. A container requesting for 10% of a system's
	// memory will have an OOM score adjust of 900. If a process in container Y
	// uses over 10% of memory, its OOM score will be 1000. The idea is that containers
	// which use more than their request will have an OOM score of 1000 and will be prime
	// targets for OOM kills.
	// Note that this is a heuristic, it won't work if a container has many small processes.
	memoryRequest := container.Resources.Requests.Memory().Value()
	oomScoreAdjust := 1000 - (1000*memoryRequest)/memoryCapacity
	// A guaranteed pod using 100% of memory can have an OOM score of 10. Ensure
	// that burstable pods have a higher OOM score adjustment.
	if int(oomScoreAdjust) < (1000 + guaranteedOOMScoreAdj) {
		return (1000 + guaranteedOOMScoreAdj)
	}
	// Give burstable pods a higher chance of survival over besteffort pods.
	if int(oomScoreAdjust) == besteffortOOMScoreAdj {
		return int(oomScoreAdjust - 1)
	}
	return int(oomScoreAdjust)
}
```
如果是guaranteed是-988 如果是besteffort则是1000，其它情况下，计算公式如上面代码所示：
oomScoreAdjust := 1000 - (1000*memoryRequest)/memoryCapacity

那算出这个值该怎么使用呢？接着看pkg/kubelet/kuberuntime/kuberuntime_container.go
```go
func (m *kubeGenericRuntimeManager) generateLinuxContainerConfig(container *v1.Container, pod *v1.Pod, uid *int64, username string) *runtimeapi.LinuxContainerConfig {
...
oomScoreAdj := int64(qos.GetContainerOOMScoreAdjust(pod, container,
		int64(m.machineInfo.MemoryCapacity)))
lc.Resources.OomScoreAdj = oomScoreAdj	
...
```
这个就是在启动容器时候使用的，pkg/kubelet/kuberuntime/kuberuntime_container.go
```go
func (m *kubeGenericRuntimeManager) startContainer(podSandboxID string, podSandboxConfig *runtimeapi.PodSandboxConfig, container *v1.Container, pod *v1.Pod, podStatus *kubecontainer.PodStatus, pullSecrets []v1.Secret, podIP string) (string, error) {
...

containerConfig, err := m.generateContainerConfig(container, pod, restartCount, podIP, imageRef)

containerID, err := m.runtimeService.CreateContainer(podSandboxID, containerConfig, podSandboxConfig)

```
作为容器的启动参数。
docker在这个[pr](https://github.com/moby/moby/pull/16277)中已经能支持 OomScoreAdj
