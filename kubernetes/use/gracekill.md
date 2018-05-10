
## 优雅关闭

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
    name: test
spec:
    replicas: 1
    template:
        spec:
            containers:
              - name: test
                image: ...
            terminationGracePeriodSeconds: 60
```

也可以通过命令行
```
kubectl delete deployment test --grace-period=60
```