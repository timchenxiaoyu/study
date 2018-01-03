
ubuntu14.04 降低内核版本到3.13.x


```shell
#!/bin/sh

KERNEL_VERSION="3.13.0-29"

sudo apt-get update
sudo aptitude install -y linux-image-${KERNEL_VERSION}-generic \
     linux-headers-${KERNEL_VERSION} linux-image-extra-${KERNEL_VERSION}-generic
     
sudo sed -i "s/GRUB_DEFAULT=.*/GRUB_DEFAULT=\"Advanced options for Ubuntu>Ubuntu, with Linux ${KERNEL_VERSION}-generic\"/" /etc/default/grub
sudo update-grub
sudo reboot

```
上面如果安装失败请尝试
```go
sudo apt-get install linux-generic
```
