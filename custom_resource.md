# **Kubernetes Custom Resource (CR) Nedir?**

Önce kavramları basitçe apaçık hale getirelim:

- **Custom Resource (CR)**, bir **Custom Resource Definition (CRD)** tarafından tanımlanan özel kaynak türünün bir örneğidir.

- Kubernets'te CRD tam olarak bir **OpenAPI v3 Schema**'dır. CR ise bu şemaya uygun olarak doldurulup API Server'a gönderilen JSON/YAML verisidir. Yani Kubernetes API Server, dışarıdan gelen bir CR (Custom Resource) isteğini `etcd` veritabanına yazmadan önce **Admission Control** aşamasında strict (katı) bir yapısal doğrulamadan geçirir. Bu doğrulamayı, CRD'nin içine gömdüğünüz OpenAPI v3 kurallarına bakarak yapar. İstediğiniz tüm kısıtlamaları (zorunlu alanlar, tip belirleme, regex, karakter sınırı) burada tanımlarsınız.

**Custom Resource (CR)**, varsayılan olarak sunulmayan, ancak ihtiyaçlara göre tanımlanabilen **özel API nesneleridir**. Kubernetes’in yerleşik kaynakları (`Pod`, `Deployment`, `Service` vb.) yeterli gelmediğinde, **kendi domain-specific nesnelerini** oluşturmanı sağlar.

> **Operator Nedir?**
> 
> Custom Resource kavramına devam etmeden önce operatörü anlayalım. **Operator**, Kubernetes’in **Custom Resource** (CR) kavramını kullanarak karmaşık uygulamaların veya servislerin yaşam döngüsünü yönetmek için geliştirilmiş bir yazılım bileşenidir. Operator kodunu go/python ile yazar bir Dockerfile ile yansı haline getirir ve DeploymentSet olarak K8s'e yüklersiniz. Operator çalışırken hedeflediği CR'nin sisteme yüklendiğini görür ve go kodunun içeriğinde ne varsa ona göre harekete geçer (pod, deployment yaratır veya başka şeyler yapar).

### Örnek Zincir

Akışımız **Kod → İmaj → Deployment → CRD/CR → Operator mantığı → Pod** zinciri şeklinde olacak. 

Buna göre tüm dosyalarımızı görelim:

```bash
echojob-operator/
├── Dockerfile                  # Operator kodunu çalıştırabilmek için konteyner imajı gerek
├── go.mod
├── go.sum
├── main.go                     # Operator giriş noktası
├── controllers/
│   └── echojob_controller.go   # Reconcile mantığı
├── config/
│   ├── crd/
│   │   └── echojob_crd.yaml    # CRD tanımı
│   ├── samples/
│   │   └── echojob_cr.yaml     # CR örneği
│   ├── manager/
│   │   └── deployment.yaml     # Operator Deployment manifesti
│   └── rbac/
│       └── rbac.yaml               # ServiceAccount, Role, RoleBinding
```

```Dockerfile
# Go tabanlı Operator için Dockerfile multi-stage şeklinde!
FROM golang:1.21 as builder

WORKDIR /workspace
# Go mod dosyalarını kopyala
COPY go.mod go.sum ./
RUN go mod download

# Kaynak kodu kopyala
COPY main.go .
COPY controllers/ ./controllers/

# Operator binary'sini derle
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o manager main.go

# Final imaj
# Yansımızın adı şu olsun: my-echojob-operator:latest
FROM gcr.io/distroless/base-debian11
WORKDIR /
COPY --from=builder /workspace/manager .
USER nonroot:nonroot

ENTRYPOINT ["/manager"]
```

Konteyner yansısı hazır!
Ancak operatörün 
- Kubernetes API'sine erişmesi,
- CRD ve CR nesnelerini okuyabilir,
- Pod/Deployment gibi kaynakları yaratabilir olması için RBAC gerekiyor!

```yaml
# ServiceAccount: Operator’un kimliği
apiVersion: v1
kind: ServiceAccount
metadata:
  name: echojob-operator-sa
  namespace: default

---
# Role: Operator’un izinleri
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: echojob-operator-role
  namespace: default
rules:
  - apiGroups: [""]                # core API group
    resources: ["pods"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: ["example.com"]     # bizim CRD grubumuz
    resources: ["echojobs"]
    verbs: ["get", "list", "watch"]

---
# RoleBinding: ServiceAccount’u Role ile bağlar
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: echojob-operator-rolebinding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: echojob-operator-sa
    namespace: default
roleRef:
  kind: Role
  name: echojob-operator-role
  apiGroup: rbac.authorization.k8s.io
```


Haydi şimdi k8s'e yükleyelim operatörü:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echojob-operator
  labels:
    app: echojob-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echojob-operator
  template:
    metadata:
      labels:
        app: echojob-operator
    spec:
      # serviceAccountName: Operator’un Kubernetes API’sine erişebilmesi için gerekli. 
      #                     Genelde RBAC ile birlikte tanımlanır.
      serviceAccountName: echojob-operator-sa
      containers:
        - name: operator
          image: my-echojob-operator:latest   # Senin build edip push ettiğin imaj
          imagePullPolicy: IfNotPresent
          resources:
            limits:
              cpu: "200m"
              memory: "256Mi"
            requests:
              cpu: "100m"
              memory: "128Mi"
```

Operatör bizim CR'leri takip edecek ama CR'lerin bir şema olan CRD'ye göre yaratılması için önce şablonu yani CRD'yi sonra CR'yi tanımlayalım:

*CRD Tanımı*
- Kubernetes’e `EchoJob` adında yeni bir kaynak tipi tanımlıyoruz.
- Bu CRD, Kubernetes API’sine yeni bir tür ekler. 
- Artık `kubectl get echojobs` gibi komutlar çalışır.

```yaml
apiVersion:apiextensions.k8s.io/v1
kind:CustomResourceDefinition
metadata:
  name:echojobs.example.com
spec:
  group:example.com
  names:
    kind:EchoJob
    plural:echojobs
  scope:Namespaced
  versions:
    -name:v1
      served:true
      storage:true
```

*Custom Resource (CR) Oluşturma*
- Sonra bu tipten bir nesne yaratıyorsun: `hello-world`. 
- Bu nesnenin içinde sadece bir mesaj var: “Merhaba Kubernetes!”. 
- Kubernetes bu nesneyi kaydeder ama kendi başına bir şey yapmaz.

```yaml
apiVersion: example.com/v1
kind: EchoJob
metadata:
  name: hello-world
spec:
  message: "Merhaba Kubernetes!"
```

*Operator Mantığı*

Cluster’da çalışan Operator (aslında bir Pod/Deployment olarak senin kurduğun yazılım) bu CR’yi izler.

-   CR içindeki `message` alanını okur.
-   Bu mesajı ekrana basacak bir Pod manifesti oluşturur.
-   Pod’u başlatır. Pod çalışır ve log’a “Merhaba Kubernetes!” yazar.

Pod manifestini Operator kendisi üretir:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: echo-hello-world
spec:
  containers:
    - name: echo
      image: busybox
      command: ["sh", "-c", "echo Merhaba Kubernetes!"]
```



### Örnekler

#### 1. `PostgreSQLTenant` objesi tasarladığımızı düşünelim

Gerçek dünya senaryolarında, özellikle veritabanı yönetimi gibi kritik altyapı süreçlerinde CRD'ler çok katı kurallarla yazılır. API Server'ın tam bir "bekçi" gibi davranmasını sağlayan, üretim ortamında kullanılabilecek seviyede katı bir CRD ve onun CR örneğini inceleyelim.

##### Makro: Veritabanı Otomasyonu (Katı Şema)

Bir Kubernetes cluster'ında, geliştirici ekiplerin kendi PostgreSQL veritabanlarını otomatik olarak yaratabilmeleri için bir `PostgreSQLTenant` objesi tasarladığımızı düşünelim. Geliştiricilerin yanlış konfigürasyon yapmasını (örneğin devasa boyutlarda disk istemesini veya veritabanı adında geçersiz karakterler kullanmasını) doğrudan API Server seviyesinde engellememiz gerekir.

##### Mikro: Katı Kurallı CRD (Şema Tanımı)

Aşağıdaki CRD, OpenAPI v3 şeması kullanarak veri tiplerini, zorunlu alanları, regex desenlerini ve alt/üst limitleri dikte eder.


```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: postgresqltenants.db.sistem.com
spec:
  group: db.sistem.com
  names:
    kind: PostgreSQLTenant
    plural: postgresqltenants
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              # KURAL 1: Bu 3 alan kesinlikle YAML'da olmak zorunda
              required: ["dbName", "owner", "storageMB"]
              properties:
                dbName:
                  type: string
                  # KURAL 2: Sadece küçük harf, rakam ve alt çizgi. Max 32 karakter.
                  pattern: '^[a-z0-9_]+$'
                  maxLength: 32
                owner:
                  type: string
                  # KURAL 3: Sadece küçük harf.
                  pattern: '^[a-z]+$'
                storageMB:
                  type: integer
                  # KURAL 4: En az 512 MB, en fazla 10 GB (10240 MB) disk istenebilir.
                  minimum: 512
                  maximum: 10240
                encoding:
                  type: string
                  # KURAL 5: Belirtilmezse UTF8 olsun. Belirtilirse sadece bu iki değerden biri olabilir.
                  default: "UTF8"
                  enum: ["UTF8", "LATIN1"]
```

##### Mikro: Geçerli Bir CR (Veri Dosyası)

Şimdi bu katı kurallardan başarıyla geçip `etcd`'ye yazılabilecek, örneğin bir RAG (Retrieval-Augmented Generation) uygulamasının vektör verilerini tutacak bir veritabanı isteği (CR) hazırlayalım.

YAML

```container
apiVersion: db.sistem.com/v1
kind: PostgreSQLTenant
metadata:
  name: vector-rag-db
  namespace: ai-apps
spec:
  dbName: rag_vectors_prod # Regex'e ve uzunluk sınırına uygun
  owner: aiservice # Sadece küçük harf, uygun
  storageMB: 4096 # 512 ile 10240 arasında, uygun
  encoding: UTF8 # Enum listesinde var, uygun
```

*Eğer `storageMB: 50000` yazsaydık veya `dbName: RAG-DB-1` (büyük harf ve tire içeriyor) yapsaydık, `kubectl apply -f` komutunu çalıştırdığımız saniye API Server isteği reddederdi.*



#### 2. Prometheus-Operator (kube-prometheus-stack)

Bu chart, k8s cluster içi izleme (monitoring) süreçlerini iyileştirir. 

> Her şey kurulu ve çalışıyor diye düşünelim. Normalde k8s içinde yeni bir uygulamanın metriklerini izlemek için Prometheus [konfigürasyon dosyası](https://prometheus.io/docs/prometheus/latest/configuration/configuration/)nı (`prometheus.yml` dosyası 3 ana bölümden oluşur: **`global`** genel ayarlar, **`scrape_configs`** metriklerin çekileceği hedefler ve **`rule_files`** kurallar) elle düzenleyip her seferinde Prometheus'u yeniden başlatmamız veya Prometheus sürecini yeniden başlatmadan HTTP API üzerinden (`/-/reload`) sıcak konfigürasyon güncellemesi (hot reload) yapmak gerekir. 

Operator mimarisi bunu CR'ler ile çözer. Prometheus Operator, Kubernetes API Server'ı sürekli dinler (watch). Yeni bir `ServiceMonitor` türünde CR yakaladığında, arkada hedef Prometheus konfigürasyonunu günceller. Prometheus pod'unun içindeki `config-reloader` adındaki sidecar container bu değişikliği fark eder ve ana Prometheus sürecini yeniden başlatmadan HTTP API üzerinden (`/-/reload`) sıcak konfigürasyon güncellemesi (hot reload) yapar.

**Mikro Yapı:** `ServiceMonitor`

Uygulamanızın metriklerini hangi porttan ve hangi path'ten toplayacağını (`/metrics`) Prometheus'a bildiren CR'dir.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: payment-api-monitor
  labels:
    release: prometheus-stack # Operator'ın bu CR'i yakalamasını sağlayan label
spec:
  selector:
    matchLabels:
      app: payment-api # Hangi Service'i izleyeceğini seçer
  endpoints:
  - port: web-metrics
    path: /metrics
    interval: 15s
```

