# kubernetes kubeconfig

下面是admin使用的kubeconfig配置文件

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: xxxx=
    server: https://localhost:6443/
  name: local-up-cluster
contexts:
- context:
    cluster: local-up-cluster
    user: local-up-cluster
  name: local-up-cluster
current-context: local-up-cluster
kind: Config
preferences: {}
users:
- name: local-up-cluster
  user:
    client-certificate-data: xxxx
    client-key-data: xxxx=

```

### certificate-authority-data
ca证书的base64加密后

### client-certificate-data
admin的crt base64加密后

### client-key-data
admin的key base64加密后


base64 加解密： http://www1.tc711.com/tool/BASE64.htm
证书验证：  http://web.chacuo.net/netssldecoder  