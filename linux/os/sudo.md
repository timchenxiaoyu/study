

ubuntu每个输入密码实在是太麻烦了，没关系

```go
cd /etc/sudoers.d   
  
sudo vi nopasswd4sudo  

//输入   
yourusername ALL=(ALL) NOPASSWD : ALL  
```

如果实在不过瘾，那就直接切换到root
```go
//不过在此之前还是需要先给root设置一个密码
sudo passwd root
su root
```