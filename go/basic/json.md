JSON

这里需要注意，这里的Namespace需要大写，不然是无法序列化的
```go
func main() {

		project := struct {
			Namespace string `json:"namespace"`
		}{
			Namespace: "chenreg",
		}

		fmt.Printf("%v \n",project)
		data, _ := json.Marshal(&project)

		fmt.Println(string(data))

	m := make(map[string]string,2)
	m["ss"]="44"
	m["sds"]="44"
	m["sdsd"]="44"
	fmt.Println(len(m))
	var s *Serverslice
	str := `{"servers":[{"serverName":"Shanghai_VPN","serverIP":"127.0.0.1"},{"serverName":"Beijing_VPN","serverIP":"127.0.0.2"}]}`
	json.Unmarshal([]byte(str), &s)
	fmt.Println(s)
}
```