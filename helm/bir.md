## Helm Chart Deposunu Eklemek

Repo komutlarına [buradan](https://helm.sh/docs/helm/helm_repo/) erişebilirsiniz.

Örnek olarak `helm repo add bitnami https://charts.bitnami.com/bitnami` komutu ile Bitnami deposunu ekleyebiliriz

```shell
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm search repo bitnami
$ helm install my-release bitnami/<chart>
```

### Chart'ı İndirmek

`helm fetch oci://registry-1.docker.io/bitnamicharts/wordpress` ile WP'nin chartını indiriyoruz.

![image](https://github.com/user-attachments/assets/a032a820-ae8f-4024-b9fe-56f804e33314)

### Chart Kurmak

`helm install [release-name] [chart] [flags]` Komutuyla helm chart kurulumu yapabiliriz [*](https://phoenixnap.com/kb/helm-install-command).

## Helm Chart Dosya Yapısı

![image](https://github.com/user-attachments/assets/831cdbf5-8f44-48df-93f2-cb0912f5c68a)

![image](https://github.com/user-attachments/assets/85b0ed36-f81a-4db8-af0e-61fb0e076f03)

![image](https://github.com/user-attachments/assets/08c9e983-66bc-496a-b1f1-9076358af111)

![image](https://github.com/user-attachments/assets/6c66ecc6-7ec5-48f7-a5c5-a34f0d0a37d4)

![image](https://github.com/user-attachments/assets/6ce56d50-a764-4408-ad12-5c4710a9041a)

## Helm Chart'ın Values.yaml Değerlerini Özelleştirmek

`helm show values <chart'ın adı>` komutuyla chart içinde `values.yml` dosyasındaki değerleri komut satırında görüntüleyebiliriz:

![image](https://github.com/user-attachments/assets/6aba199c-2e69-48ef-876f-e7655b055625)

İndirdiğimiz chart'ın values.yml içeriğine bakalım:

![image](https://github.com/user-attachments/assets/ff6814b5-9071-493c-b28f-21ab32faa74f)

Satır içi bir anahtarın değerini değiştirmek için `--set <anahtar>=<deger>` komutunu CLI'da kullanırız. 

```sh
helm install surum-adim ./wordpress --dry-run --set image.tag="CEM"
```

![image](https://github.com/user-attachments/assets/a3bab1d4-6fba-47b7-988c-4cd2f275870c)

Ayrıca bir yaml dosyası hazırlayıp `-f <dosya_adı.yml>` bayrağı ile `values.yml` içindeki değerleri ezebiliriz.

`ezen.yml` 
```yaml
image:
  tag: CEM
```

```sh
helm install surum-adim ./wordpress -f ./ezen.yml --dry-run
```






