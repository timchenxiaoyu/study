## cert认证
证书认证是k8s最常用也是比较安全的一种方式，通过--client-ca-file=SOMEFILE方式开启
怎样生成证书不在此处赘述，通过很多工具都可以如openssl等。
看k8s的证书认证代码实现，pkg/kubeapiserver/authenticator/config.go
```go
func newAuthenticatorFromClientCAFile(clientCAFile string) (authenticator.Request, error) {
	roots, err := certutil.NewPool(clientCAFile)
	if err != nil {
		return nil, err
	}

	opts := x509.DefaultVerifyOptions()
	opts.Roots = roots

	return x509.New(opts, x509.CommonNameUserConversion), nil
}
```
读取ca证书，k8s.io/client-go/util/cert/io.go
```go
func NewPool(filename string) (*x509.CertPool, error) {
	certs, err := CertsFromFile(filename)
	if err != nil {
		return nil, err
	}
	pool := x509.NewCertPool()
	for _, cert := range certs {
		pool.AddCert(cert)
	}
	return pool, nil
}
```
和其它认证一样apiserver/pkg/authentication/request/x509/x509.go
```go
func New(opts x509.VerifyOptions, user UserConversion) *Authenticator {
	return &Authenticator{opts, user}
}
```
还有一个认证的方法
```go
func (a *Authenticator) AuthenticateRequest(req *http.Request) (user.Info, bool, error) {
	if req.TLS == nil || len(req.TLS.PeerCertificates) == 0 {
		return nil, false, nil
	}

	// Use intermediates, if provided
	optsCopy := a.opts
	if optsCopy.Intermediates == nil && len(req.TLS.PeerCertificates) > 1 {
		optsCopy.Intermediates = x509.NewCertPool()
		for _, intermediate := range req.TLS.PeerCertificates[1:] {
			optsCopy.Intermediates.AddCert(intermediate)
		}
	}

	chains, err := req.TLS.PeerCertificates[0].Verify(optsCopy)
	if err != nil {
		return nil, false, err
	}

	var errlist []error
	for _, chain := range chains {
		user, ok, err := a.user.User(chain)
		if err != nil {
			errlist = append(errlist, err)
			continue
		}

		if ok {
			return user, ok, err
		}
	}
	return nil, false, utilerrors.NewAggregate(errlist)
}
```


通过相同的ca签名的证书，双向认证即可通过
```go
openssl genrsa -out test-key.pem 2048
openssl req -new -key test-key.pem -subj "/CN=tim" -out test.csr
openssl x509 -req -in test.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out test.pem -days 5000
curl -v --cacert /etc/kubernetes/pki/ca.pem --cert /tmp/test.pem --key /tmp/test-key.pem  https://10.39.0.105:6443/
curl -X GET --cacert /etc/kubernetes/ssl/ca.pem --cert /etc/kubernetes/ssl/test.crt --key /etc/kubernetes/ssl/test-key.crt --tlsv1.2
 -H "Content-Type: application/json" https://master:6443/api/v1/pods
```