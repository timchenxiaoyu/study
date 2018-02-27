## token 认证
在kubernetes里面通过 --token-auth-file=/etc/kubernetes/pki/tokens.csv设定token认证，下面我们从源码角度是去看它是如何实现的，
```go
	if len(config.TokenAuthFile) > 0 {
		tokenAuth, err := newAuthenticatorFromTokenFile(config.TokenAuthFile)
		if err != nil {
			return nil, nil, err
		}
		authenticators = append(authenticators, tokenAuth)
		hasTokenAuth = true
	}
```
如果设置了token-auth-file将会开启token认证，但k8s里面没有像其他的web系统通过用户uid或者时间md5得带token，而是直接通过文件获取
所以我们必须生成一个cvs格式的token文件,结构很简单，token, user name, user uid, user group,下面是一个例子
```go
bf8cb8725efab8c4,kubeadm-node-csr,de1244f4-4c0d-11e7-a252-52549da43ad9,system:kubelet-bootstrap
```
当apiservice启动时候读取这个文件apiserver/pkg/authentication/token/tokenfile/tokenfile.go
```go
func NewCSV(path string) (*TokenAuthenticator, error) {
	file, err := os.Open(path)
	if err != nil {
		return nil, err
	}
	defer file.Close()

	recordNum := 0
	tokens := make(map[string]*user.DefaultInfo)
	reader := csv.NewReader(file)
	reader.FieldsPerRecord = -1
	for {
		record, err := reader.Read()
		if err == io.EOF {
			break
		}
		if err != nil {
			return nil, err
		}
		if len(record) < 3 {
			return nil, fmt.Errorf("token file '%s' must have at least 3 columns (token, user name, user uid), found %d", path, len(record))
		}
		obj := &user.DefaultInfo{
			Name: record[1],
			UID:  record[2],
		}
		recordNum++
		if _, exist := tokens[record[0]]; exist {
			glog.Warningf("duplicate token has been found in token file '%s', record number '%d'", path, recordNum)
		}
		tokens[record[0]] = obj

		if len(record) >= 4 {
			obj.Groups = strings.Split(record[3], ",")
		}
	}

	return &TokenAuthenticator{
		tokens: tokens,
	}, nil
}
```
返回tokens 这个map，key是token value是用户名和用户id组合的struct。
这样就可以创建一个token认证的apiserver/pkg/authentication/request/bearertoken/bearertoken.go
```go
type Authenticator struct {
	auth authenticator.Token
}

func New(auth authenticator.Token) *Authenticator {
	return &Authenticator{auth}
}
```
当请求过来时，
```go
func (a *Authenticator) AuthenticateRequest(req *http.Request) (user.Info, bool, error) {
	auth := strings.TrimSpace(req.Header.Get("Authorization"))
	if auth == "" {
		return nil, false, nil
	}
	parts := strings.Split(auth, " ")
	if len(parts) < 2 || strings.ToLower(parts[0]) != "bearer" {
		return nil, false, nil
	}

	token := parts[1]

	// Empty bearer tokens aren't valid
	if len(token) == 0 {
		return nil, false, nil
	}

	user, ok, err := a.auth.AuthenticateToken(token)
	// if we authenticated successfully, go ahead and remove the bearer token so that no one
	// is ever tempted to use it inside of the API server
	if ok {
		req.Header.Del("Authorization")
	}

	// If the token authenticator didn't error, provide a default error
	if !ok && err == nil {
		err = invalidToken
	}

	return user, ok, err
}
```
获取用户传入的token，AuthenticateToken方法去认证apiserver/pkg/authentication/token/tokenfile/tokenfile.go
```go
func (a *TokenAuthenticator) AuthenticateToken(value string) (user.Info, bool, error) {
	user, ok := a.tokens[value]
	if !ok {
		return nil, false, nil
	}
	return user, true, nil
}
```
就是遍历map查找相同的token，这样就完成了token的认证
那么可以通过,去验证token，当然通过-k可以跳过证书认证的部分
```go
curl -X GET --cacert /etc/kubernetes/ssl/ca.pem --tlsv1.2 -H "Content-Type: application/json"  -H "Authorization: Bearer abc" https://master:6443/api/v1/pods
```