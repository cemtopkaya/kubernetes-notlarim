##### Host ve Konteyner Aynı Kernel'i Kullanıyor
> Host işletim sisteminizi öğrenmek için `uname -a` komutunu çalıştırın ve kernel sürümünü öğrenin.
```shell
$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.6 LTS"

$ uname -a
Linux ip-10-0-0-100 5.4.0-1045-aws #47~18.04.1-Ubuntu SMP Tue Apr 13 15:58:14 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```
![image](https://user-images.githubusercontent.com/261946/221383366-6b09e8ca-cd4a-467c-a50d-a0f794cf3ba8.png)


> Şimdi Ubuntu sürümlerinin yansılarını çekelim (docker yerine containerd olduğu için docker.hub üstündeki yansıyı çekmek için komut `$ sudo ctr image pull docker.io/library/ubuntu:focal`)

![image](https://user-images.githubusercontent.com/261946/221383305-282bf9e6-b35d-45db-97f9-6c35c71f9313.png)

> Listeleyelim yansıları 

![image](https://user-images.githubusercontent.com/261946/221383630-c648610c-4c2b-4a97-a4b0-c6bce225445b.png)

> ve hepsinden bir konteyner yaratıp konteyner içinde `uname -a` çalıştıralım:
```shell
$ ctr run --rm -t docker.io/library/ubuntu:bionic ccc-ubuntu uname -a
```

![image](https://user-images.githubusercontent.com/261946/221383547-f1be2733-aa97-4a9e-81e2-78532d8c1a0f.png)
