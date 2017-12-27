### 1. 判断java版本
```
java -version 

```

### 2. 安装maven
```
cd /usr/local
wget http://www-eu.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
sudo tar xzf apache-maven-3.5.2-bin.tar.gz
sudo ln -s apache-maven-3.5.2  maven

```
### 3. 设置环境变量
```
sudo vi /etc/profile.d/maven.sh
export M2_HOME=/usr/local/maven
export PATH=${M2_HOME}/bin:${PATH}
source /etc/profile.d/maven.sh

```
### 4. 检查maven版本
```
mvn -version 
```
### 5. 清理环境
```
rm -f /usr/local/apache-maven-3.5.2-bin.tar.gz

```