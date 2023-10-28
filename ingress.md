# GİRİŞ (INGRESS)

![image](https://github.com/cemtopkaya/kubernetes-notlarim/assets/261946/c41dae9f-4738-4891-9f7e-70cbd2816910)

Kubernetes Ingress nesnesi, bir küme içindeki hizmetlere harici erişimi yönetmek için kullanılan bir kaynaktır. Tipik olarak Giriş kaynakları, HTTP(S) yollarını ortaya çıkarmak için kullanılır. Bir Giriş kaynağı aşağıdakilerden herhangi birini sağlayabilir:

* Yük dengeleme
* SSL sonlandırma
* HTTP Sanal Barındırma (bir istekte alan adına göre yönlendirme)
* HTTP URL yolu tabanlı yönlendirme

Yük Dengeleyici'nin En Üstten Görünümü:
![image](https://user-images.githubusercontent.com/261946/230696683-cbfaf67c-b9d4-4e31-8e9d-d19e43654648.png)

Bir giriş (ingress) kaynağının kullanılması, bir giriş Denetleyicisinin Kubernetes kümesine yüklenmesini ve çalışmasını gerektirir. 
Birçok [ingress denetleyicisi](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/#additional-controllers) türü vardır. 

Mevcut en popüler giriş denetleyicilerinden bazıları şunlardır:
* NGINX
* Traefik
* Haproxy
* Envoy

 NGINX giriş denetleyicisi bir Ağ Yük Dengeleyici (Load balancer) oluşturacaktır. 
 NGINX Giriş Denetleyicisi, Google Cloud Platform (GCP) ve Microsoft Azure gibi diğer bulutları destekler. 
 Ayrıca, çıplak donanım kümesine (bare-metal cluster) konuşlandırmayı da destekler; bu durumda, harici trafiği almak için Düğüm Bağlantı Noktasına (Node Port) sahip bir Hizmet kaynağı (service resource) oluşturur .
 
 
 Aşağıda bir hizmet noktası ve bunun kullanıldığı POD yaratılma kodu var. 
 red-app adında bir pod ile `<url>/red` adresine gelen isteği bu poda, `<url>/blue` isteğini de aşağıda yaratacağımız blue-app poduna yönlendireceğiz.
 
 ```shell
$ cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red-app
spec:
  selector:
    matchLabels:
      app: red-app
  replicas: 2
  template:
    metadata:
      labels:
        app: red-app
    spec:
      containers:
      - name: red-app
        image: public.ecr.aws/cloudacademy-labs/cloudacademy/labs/k8s-ingress-app:f9a36c8
        env:
        - name: COLOR
          value: '#FAA0A0'
        ports:
        - containerPort: 8000
---
apiVersion: v1
kind: Service
metadata:
  name: red-app
spec:
  selector:
    app: red-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
EOF
```

Şimdi blue-app podunu yaratalım:
```shell
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-app
spec:
  selector:
    matchLabels:
      app: blue-app
  replicas: 2
  template:
    metadata:
      labels:
        app: blue-app
    spec:
      containers:
      - name: blue-app
        image: public.ecr.aws/cloudacademy-labs/cloudacademy/labs/k8s-ingress-app:f9a36c8
        env:
        - name: COLOR
          value: '#A7C7E7'
        ports:
        - containerPort: 8000
---
apiVersion: v1
kind: Service
metadata:
  name: blue-app
spec:
  selector:
    app: blue-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
EOF
```

Çalıştıktan sonra hizmet ve pod'u görebiliriz:

![image](https://user-images.githubusercontent.com/261946/230256699-146d80d0-ee6b-40bf-b751-8d1372d91be7.png)

Şimdi giriş yolu ayarları yaparak web isteklerini pod'a yönlendireceğiz: 

```yaml
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lab-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /blue
        pathType: Prefix
        backend:
          service:
            name: blue-app
            port:
              number: 80
      - path: /red
        pathType: Prefix
        backend:
          service:
            name: red-app
            port:
              number: 80
EOF
```

Oluşturduğumuz giriş deneteleyicisine gelen isteklerin günlük kayıtlarını görmek için:

```shell
kubectl logs -l app.kubernetes.io/name=ingress-nginx --namespace ingress-nginx
```

Uygulamalara erişimi alan adı yoluyla ağ içinden yapmak istersek aşağodaki komutu çalıştırarak ingress ayarlarını değiştireceğiz:

```shell
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lab-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: blue.example.com
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: blue-app
            port:
              number: 80
  - host: red.example.com
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: red-app
            port:
              number: 80
EOF
```

Ve isteklerimizi yapınca:
![image](https://user-images.githubusercontent.com/261946/230258923-2c9ab6dc-93b9-4313-af65-219ac0d64b6e.png)




