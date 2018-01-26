# jwt golang实现

先介绍一个开源的jwt实现，github.com/dgrijalva/jwt-go
现在我们通过这个库去实现，先看这个库的实现
```go
func NewWithClaims(method SigningMethod, claims Claims) *Token {
	return &Token{
		Header: map[string]interface{}{
			"typ": "JWT",
			"alg": method.Alg(),
		},
		Claims: claims,
		Method: method,
	}
}

func (t *Token) SignedString(key interface{}) (string, error) {
	var sig, sstr string
	var err error
	if sstr, err = t.SigningString(); err != nil {
		return "", err
	}
	if sig, err = t.Method.Sign(sstr, key); err != nil {
		return "", err
	}
	return strings.Join([]string{sstr, sig}, "."), nil
}
```
就是jwt定义的实现，只不过Claims就是上面playload（负载）。
那么我们使用就更加简单了
```go
func createAuthTokenString(uuid string, role string, csrfSecret string) (authTokenString string, err error) {
	authTokenExp 	:= time.Now().Add(models.AuthTokenValidTime).Unix()
	authClaims 		:= models.TokenClaims{
		jwt.StandardClaims{
			Subject: uuid,
			ExpiresAt: authTokenExp,
		},
		role,
		csrfSecret,
	}

	// create a signer for rsa 256
	authJwt := jwt.NewWithClaims(jwt.GetSigningMethod("RS256"), authClaims)

	// generate the auth token string
	authTokenString, err = authJwt.SignedString(signKey)
	return
}
```
这样就生成了一个authToken的token了。客户端拿到这个token之后就就已经就可以解析出用户的角色等其他信息了。

当用户登录时候,验证用户名和密码后生成authToken和refreshToken
```go
authTokenString, refreshTokenString, csrfSecret, err := myJwt.CreateNewTokens(uuid, user.Role)
setAuthAndRefreshCookies(&w, authTokenString, refreshTokenString)
w.Header().Set("X-CSRF-Token", csrfSecret)
w.WriteHeader(http.StatusOK)
```
并返回到客户端

当用户再调用其它接口时候，先获取到这两个token，然后验证其有限性
```go
authTokenString, refreshTokenString, csrfSecret, err := myJwt.CheckAndRefreshTokens(AuthCookie.Value, RefreshCookie.Value, requestCsrfToken)

```
具体实现如下，verifyKey就是公钥
```go
authToken, err := jwt.ParseWithClaims(oldAuthTokenString, &models.TokenClaims{}, func(token *jwt.Token) (interface{}, error) {
		return verifyKey, nil
	})
	if authToken.Valid {
    		log.Println("Auth token is valid")
    		//更新refresh token过期时间
    		newRefreshTokenString, err = updateRefreshTokenExp(oldRefreshTokenString)
    }		
```
并且返回token到客户端



当用户退出的时候,清空token
```go
	authCookie := http.Cookie{
		Name: "AuthToken",
		Value: "",
		Expires: time.Now().Add(-1000 * time.Hour),
		HttpOnly: true,
	}

	http.SetCookie(*w, &authCookie)

	refreshCookie := http.Cookie{
		Name: "RefreshToken",
		Value: "",
		Expires: time.Now().Add(-1000 * time.Hour),
		HttpOnly: true,
	}

	http.SetCookie(*w, &refreshCookie)
```
参考：github.com/adam-hanna/goLang-jwt-auth-example