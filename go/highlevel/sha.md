是一个密码散列函数家族，是FIPS所认证的安全散列算法。能计算出一个数字消息所对应到的，长度固定的字符串（又称消息摘要）的算法。
且若输入的消息不同，它们对应到不同字符串的机率很高。
下面是golang完整的加密解密、签名和验证的完整示例：
```go
package main

import (
	"encoding/pem"
	"encoding/base64"
	"crypto/x509"
	"crypto/rsa"
	"crypto/rand"
	"errors"
	"crypto"
	"io"
	"os"
	"bytes"
	"encoding/asn1"
	"io/ioutil"
	"fmt"
)



const (
	RSA_ALGORITHM_SIGN = crypto.SHA256
)

type XRsa struct {
	publicKey *rsa.PublicKey
	privateKey *rsa.PrivateKey
}

// 生成密钥对
func CreateKeys(publicKeyWriter, privateKeyWriter io.Writer, keyLength int) error {
	// 生成私钥文件
	privateKey, err := rsa.GenerateKey(rand.Reader, keyLength)
	if err != nil {
		return err
	}
	derStream := MarshalPKCS8PrivateKey(privateKey)
	block := &pem.Block{
		Type:  "PRIVATE KEY",
		Bytes: derStream,
	}
	err = pem.Encode(privateKeyWriter, block)
	if err != nil {
		return err
	}
	// 生成公钥文件
	publicKey := &privateKey.PublicKey
	derPkix, err := x509.MarshalPKIXPublicKey(publicKey)
	if err != nil {
		return err
	}
	block = &pem.Block{
		Type:  "PUBLIC KEY",
		Bytes: derPkix,
	}

	err = pem.Encode(publicKeyWriter, block)
	if err != nil {

		return err

	}
	return nil

}
//创建加密工具实体
func NewXRsa(publicKey []byte, privateKey []byte) (*XRsa, error) {
	block, _ := pem.Decode(publicKey)
	if block == nil {
		return nil, errors.New("public key error")
	}
	pubInterface, err := x509.ParsePKIXPublicKey(block.Bytes)
	if err != nil {
		return nil, err
	}

	pub := pubInterface.(*rsa.PublicKey)
	block, _ = pem.Decode(privateKey)
	if block == nil {
		return nil, errors.New("private key error!")
	}

	priv, err := x509.ParsePKCS8PrivateKey(block.Bytes)
	if err != nil {
		return nil, err
	}
	pri, ok := priv.(*rsa.PrivateKey)
	if ok {
		return &XRsa {
			publicKey: pub,
			privateKey: pri,

		}, nil
	} else {
		return nil, errors.New("private key not supported")

	}

}

// 公钥加密
func (r *XRsa) PublicEncrypt(data string) (string, error) {
	partLen := r.publicKey.N.BitLen() / 8 - 11
	chunks := split([]byte(data), partLen)
	buffer := bytes.NewBufferString("")
	for _, chunk := range chunks {
		bytes, err := rsa.EncryptPKCS1v15(rand.Reader, r.publicKey, chunk)
		if err != nil {
			return "", err
		}
		buffer.Write(bytes)
	}
	return base64.RawURLEncoding.EncodeToString(buffer.Bytes()), nil

}

// 私钥解密
func (r *XRsa) PrivateDecrypt(encrypted string) (string, error) {
	partLen := r.publicKey.N.BitLen() / 8
	raw, err := base64.RawURLEncoding.DecodeString(encrypted)
	chunks := split([]byte(raw), partLen)
	buffer := bytes.NewBufferString("")
	for _, chunk := range chunks {
		decrypted, err := rsa.DecryptPKCS1v15(rand.Reader, r.privateKey, chunk)
		if err != nil {
			return "", err
		}
		buffer.Write(decrypted)
	}
	return buffer.String(), err
}

// 数据加签
func (r *XRsa) Sign(data string) (string, error) {
	h := RSA_ALGORITHM_SIGN.New()
	h.Write([]byte(data))
	hashed := h.Sum(nil)
	sign, err := rsa.SignPKCS1v15(rand.Reader, r.privateKey, RSA_ALGORITHM_SIGN, hashed)
	if err != nil {
		return "", err
	}

	return base64.RawURLEncoding.EncodeToString(sign), err
}


// 数据验签
func (r *XRsa) Verify(data string, sign string) error {
	h := RSA_ALGORITHM_SIGN.New()
	h.Write([]byte(data))
	hashed := h.Sum(nil)

	decodedSign, err := base64.RawURLEncoding.DecodeString(sign)
	if err != nil {
		return err
	}
	return rsa.VerifyPKCS1v15(r.publicKey, RSA_ALGORITHM_SIGN, hashed, decodedSign)

}


func MarshalPKCS8PrivateKey(key *rsa.PrivateKey) []byte {
	info := struct {
		Version             int
		PrivateKeyAlgorithm []asn1.ObjectIdentifier
		PrivateKey          []byte
	}{}
	info.Version = 0
	info.PrivateKeyAlgorithm = make([]asn1.ObjectIdentifier, 1)
	info.PrivateKeyAlgorithm[0] = asn1.ObjectIdentifier{1, 2, 840, 113549, 1, 1, 1}
	info.PrivateKey = x509.MarshalPKCS1PrivateKey(key)

	k, _ := asn1.Marshal(info)

	return k

}

func split(buf []byte, lim int) [][]byte {
	var chunk []byte
	chunks := make([][]byte, 0, len(buf)/lim+1)
	for len(buf) >= lim {
		chunk, buf = buf[:lim], buf[lim:]
		chunks = append(chunks, chunk)
	}

	if len(buf) > 0 {
		chunks = append(chunks, buf[:len(buf)])
	}

	return chunks

}
func init() {
//创建公私钥
	privatef,_ := os.Create("privatekey.crt")
	publicf ,_ :=os.Create("publickey.crt")

	CreateKeys(publicf,privatef,1024)
}

func main() {
	privatef,_ := os.Open("privatekey.crt")
	publicf ,_ :=os.Open("publickey.crt")



	privtebyte, _ := ioutil.ReadAll(privatef)
	publicbyte, _ := ioutil.ReadAll(publicf)
	fmt.Println(string(publicbyte))
	fmt.Println(string(privtebyte))
	rsa ,_ := NewXRsa(publicbyte,privtebyte)
    //加密
	encrptedcode,_ := rsa.PublicEncrypt("abc")
	fmt.Println(encrptedcode)
	//解密
	rwacode ,_ := rsa.PrivateDecrypt(encrptedcode)
	fmt.Println(rwacode)

	fmt.Println("*******************")

    //添加签名
	singedcode,_ := rsa.Sign("abc")
	fmt.Println(singedcode)
	//验证签名
	err := rsa.Verify("abc",singedcode)
	if err != nil{
		fmt.Println(err)
	}else{
		fmt.Println("ok")
	}


}
```