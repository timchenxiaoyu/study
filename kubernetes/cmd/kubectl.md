### 查询结果包含label

```go
kubectl get node --show-labels
```

### 导出go-template处理
```go
kubectl get no -o go-template='{{range .items}}{{if .spec.unschedulable}}{{.metadata.name}} {{.spec.externalID}}{{"\n"}}{{end}}{{end}}'
```

### 强制删除
```go
kubectl delete pods <pod> --grace-period=0 --force
```