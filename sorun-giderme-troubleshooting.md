# Sorunlar ve Cevapları

### Sorun: "The connection to the server localhost:8080 was refused - did you specify the right host or port?"
#### Çözüm
İçinde doğru yapılandırmaya sahip bir .kube dizininin olmamasıdır. `/etc/kubernetes/admin.conf` dosyasını kullanıcının ev dizininde `.kube` adında bir dizin yaratarak içine kopyalayalım.

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

![image](https://user-images.githubusercontent.com/261946/221379172-cb895eea-761c-4219-95d6-179cec394e6c.png)

![image](https://user-images.githubusercontent.com/261946/221379645-9bc062a7-795d-4eec-a315-333c8ca66508.png)

