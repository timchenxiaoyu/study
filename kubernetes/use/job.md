## 批量任务

### 非并行Job
通常创建一个Pod直至其成功结束
### 固定结束次数的Job
设置.spec.completions，创建多个Pod，直到.spec.completions个Pod成功结束
### 带有工作队列的并行Job
设置.spec.Parallelism但不设置.spec.completions，当所有Pod结束并且至少一个成功时，Job就认为是成功



|Job类型|	使用示例|	行为	|completions	|Parallelism|
| :------:| :------: | :------: | :------: | :------: |
|一次性Job|	数据库迁移|	创建一个Pod直至其成功结束|	1|	1|
|固定结束次数的Job|	处理工作队列的Pod|	依次创建一个Pod运行直至completions个成功结束	|2+|	1|
|固定结束次数的并行Job|	多个Pod同时处理工作队列|	依次创建多个Pod运行直至completions个成功结束	|2+|	2+|
|并行Job|	多个Pod同时处理工作队列|	创建一个或多个Pod直至有一个成功结束|	1|	2+|



## 定时任务cronjob
定时的任务
```go
kubectl run hello --schedule="*/1 * * * *" --restart=OnFailure --image=busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
cronjob "hello" created
```