# AUFS

aufs的全称是advanced multi-layered unification filesystem，主要功能是把多个文件夹的内容合并到一起，提供一个统一的视图，
主要用于各个Linux发行版的livecd中，以及docker里面用来组织image。

实践：
## 只读挂载
```go
# echo dir0 > dir0/001.txt
# echo dir0 > dir0/002.txt
# echo dir1 > dir1/002.txt
# echo dir1 > dir1/003.txt

# tree
.
├── dir0
│   ├── 001.txt
│   └── 002.txt
├── dir1
│   ├── 002.txt
│   └── 003.txt
└── root

# sudo mount -t aufs -o br=./dir0=ro:./dir1=ro none ./root
# cd root/
# ls
001.txt  002.txt  003.txt

# cat 002.txt 
dir0

```
从上面的演示可以看出，我们可以跳过挂载点直接读写底层的目录，这样就不受aufs的控制，但我们修改的内容（dir1里面创建的004.txt）
还是能在挂载点下看到，这是因为aufs在访问文件时，默认的做法是如果最上层目录里面没这个文件，就一层一层的往下找，所以下层有变动的话，
aufs会自动发现。

## 读写挂载
```
# mount -t aufs -o br=./dir0=rw:./dir1=ro none ./root
# echo "root->write" >> ./root/001.txt
# echo "root->write" >> ./root/002.txt
# echo "root->write" >> ./root/003.txt
# echo "root->write" >> ./root/005.txt

# ls root/
001.txt  002.txt  003.txt  005.txt

# ls dir0/
001.txt  002.txt  003.txt  005.txt

# ls dir1/
002.txt  003.txt

# cat root/002.txt 
dir0
root->write

# cat dir0/002.txt 
dir0
root->write

# cat dir1/002.txt 
dir1



```
所以dir0下的内容变化了，但dir1的内容没有任何变化

## 删除
```go
# rm ./root/001.txt ./root/002.txt ./root/003.txt ./root/005.txt
# ls dir0
#
# ls dir1
002.txt  003.txt

```