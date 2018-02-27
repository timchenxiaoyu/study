## basic 认证
basic就是用户名和密码的认证
这个和之前介绍的的认证的体系结构是一样的，先通过--basic-auth-file=SOMEFILE开启认证他需要的cvs格式是一样的
```go
password,user,uid,"group1,group2,group3"
```
就是把明文的密码写到这个文件中，如果用户请求的时候通过，header里面添加Authorization Basic BASE64ENCODED(USER:PASSWORD)的方式请求
下面看看k8s具体的实现apiserver/plugin/pkg/authenticator/password/passwordfile/passwordfile.go
```go
func NewCSV(path string) (*PasswordAuthenticator, error) {
	file, err := os.Open(path)
	if err != nil {
		return nil, err
	}
	defer file.Close()

	recordNum := 0
	users := make(map[string]*userPasswordInfo)
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
			return nil, fmt.Errorf("password file '%s' must have at least 3 columns (password, user name, user uid), found %d", path, len(record))
		}
		obj := &userPasswordInfo{
			info:     &user.DefaultInfo{Name: record[1], UID: record[2]},
			password: record[0],
		}
		if len(record) >= 4 {
			obj.info.Groups = strings.Split(record[3], ",")
		}
		recordNum++
		if _, exist := users[obj.info.Name]; exist {
			glog.Warningf("duplicate username '%s' has been found in password file '%s', record number '%d'", obj.info.Name, path, recordNum)
		}
		users[obj.info.Name] = obj
	}

	return &PasswordAuthenticator{users}, nil
}

```
和token认证一样也有一个NewCSV的解析方法，读取到users这个map，只不过key是用户名，密码是用户名用户id以及密码用户组的组合struct。
下面继续看它的认证方法apiserver/plugin/pkg/authenticator/request/basicauth/basicauth.go
```go
func (a *Authenticator) AuthenticateRequest(req *http.Request) (user.Info, bool, error) {
	username, password, found := req.BasicAuth()
	if !found {
		return nil, false, nil
	}

	user, ok, err := a.auth.AuthenticatePassword(username, password)

	// If the password authenticator didn't error, provide a default error
	if !ok && err == nil {
		err = errInvalidAuth
	}

	return user, ok, err
}
```
上面的BasicAuth的实现就去解析请求头认证,auth := r.Header.Get("Authorization")
通过AuthenticatePassword去认证apiserver/plugin/pkg/authenticator/password/passwordfile/passwordfile.go
```go
func (a *PasswordAuthenticator) AuthenticatePassword(username, password string) (user.Info, bool, error) {
	user, ok := a.users[username]
	if !ok {
		return nil, false, nil
	}
	if user.password != password {
		return nil, false, nil
	}
	return user.info, true, nil
}

```
获取相同的用户，再通过比较密码判定。
通过curl去验证
```go
echo -n "cyli:123456" | base64 会生成一串字符串，假设为fadfad

curl -X GET --cacert /etc/kubernetes/ssl/ca.pem --tlsv1.2 -H "Content-Type: application/json"  -H "Authorization: Basic fadfad" https://master:6443/api/v1/pods
```