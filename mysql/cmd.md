
### 数据导入

```
mysql -u用户名 -p密码 -h 主机地址 <导出文件名
eg:
mysql -uroot -proot -h 10.39.14.11 <file.sql

```

### 数据导出
如果表名不写则导出整个数据库
```
mysqldump -u 用户名 -p密码 -h主机名 数据库名 表名> 导出的文件名
eg:
mysqldump -u root -proot test users> wcnc_users.sql

```