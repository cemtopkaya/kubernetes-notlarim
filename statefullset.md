# StatefulSet [*](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/)

![image](https://github.com/cemtopkaya/kubernetes-notlarim/assets/261946/b80aeaeb-4186-4dce-a221-a30047d74fbc)

# Ne Nedir?

Önce temel terimlere bir göz atalım:

1. **State (Durum)**: Bir uygulamanın veya sistem bileşeninin içinde bulunduğu anlık koşul veya durum. Verilerin, yapılandırmaların, işlem durumlarının ve diğer değişkenlerin belirli bir zamandaki durumu.

2. **Stateless (Durumsuz)**: Durumsuz, bir sistem veya uygulamanın çalışma durumunun tutulmadığı veya saklanmadığı bir durumu ifade eder. Her işlem veya istek, herhangi bir önceki işlem veya istekin sonucunu veya durumunu bilmeyen ve etkilemeyen bir şekilde işlenir. Örneğin, HTTP protokolü stateless bir protokoldür.
```csharp
public class Calculator {
    public int Add(int a, int b) {
        return a + b;
    }

    public int Subtract(int a, int b) {
        return a - b;
    }
}
```
Yukarıdaki Calculator sınıfı stateless (durumsuz, veri tutmayan) yapıdadır.

3. **Stateful (Durum Taşıyan)**: Durum taşıyan, bir sistem veya uygulamanın çalışma durumunun takip edildiği ve saklandığı bir durumu ifade eder. Her işlem veya istek, önceki işlemlerin sonuçlarına veya durumuna bağlı olarak işlenir. Örneğin, bir veritabanı, durum taşıyan bir sistemdir çünkü verilerin durumu sürekli olarak güncellenir ve saklanır.
```csharp
public class Counter {
    private int count;

    public Counter() {
        count = 0;
    }

    public void Increment() {
        count++;
    }

    public void Decrement() {
        count--;
    }

    public int GetCount() {
        return count;
    }
}

```
Yukarıdaki `Counter` sınıfı stateful yapıdadır. Her bir örneğinin `count` adında bir durumu (state) vardır ve count değerini daima içinde saklar.

StatefulSet, bu "durum taşıyan" uygulamaları Kubernetes ortamında dağıtmak ve yönetmek için kullanılan bir Kubernetes kaynağıdır. 
Kubernetes'de StatefulSet, veri tabanları, mesaj kuyrukları ve diğer stateful iş yükleri gibi **içinde bulundukları durumu koruyan uygulamalar**ın Kubernetes üzerinde nasıl yönetileceğini tanımlar. 
StatefulSet, her bir pod'a benzersiz bir isim ve ağ kimliği verir ve bu pod'ların sıralı başlatılmasını ve durdurulmasını sağlar.
Yani, StatefulSet içindeki pod'lar "stateful" olarak yönetilirler. Her bir pod, kendi içindeki verileri (durumu) saklar ve bu veriler genellikle kalıcı bir depolama biriminde bulunur. 
Bu, verilerin kalıcı ve benzersiz olduğu anlamına gelir ve her pod'ın kendi durumunu sürdürdüğü anlamına gelir.
Örneğin, bir veritabanı uygulaması için StatefulSet kullanıyorsak, her bir pod kendi veritabanı örneğini içerir ve bu veritabanı örneği verileri saklar. 

> Kubernetes (k8s), stateful olmayan (stateless) uygulamaların tanımlanması için özellikle **Deployments** ve **ReplicationControllers** ve **Pod** gibi kaynaklar kullanmanızı sağlar.
> Stateless uygulamalar, her bir çalışan örneğin (pod), uygulama için aynıdır ve herhangi bir özel durumu (state) saklamazlar.

StatefulSet, 
- her bir pod'a benzersiz bir isim ve ağ kimliği verir,
- bu pod'ların sıralı başlatılmasını ve durdurulmasını sağlar,
- volume yönetimi sunar ve veritabanları gibi stateful iş yüklerinin gereksinimlerini karşılar.

Bu sayede, stateful uygulamaları Kubernetes üzerinde çalıştırmak daha kolay hale gelir.

# Basit Bir StatefulSet Örneği

Aşağıda basit bir StatefulSet YAML örneği bulunmaktadır:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```

**Başlık Servis (Headless Service) Tanımı:**
- `apiVersion: v1` ve `kind: Service` ile bir başlık servisi (headless service) tanımlanır.
- Servis adı "nginx" olarak belirtilmiştir.
- Servise `"app: nginx"` etiketini ekler. Bu etiket, başlık servisi ile pod'ları eşleştiren bir etiketleme şemasıdır.
- Servis, 80 numaralı bir portu açar ve "web" adıyla etiketler.
- `clusterIP: None` ile başlık servisi (headless service) olarak ayarlanır. Aşağıda yaratılacak StatefulSet, bu başlık servisini hedef alarak her pod'un kendi benzersiz DNS girdilerine sahip olmasını sağlar.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

Bu YAML dosyası, bir başlık servisi (Headless Service) ve bu başlık servisini hedef alan bir StatefulSet tanımı içerir. Başlık servisi, "nginx" adında etiketlenmiş pod'lara sahip bir "app: nginx" etiketleme şeması ile ilgilenir. Ayrıca bu pod'lar için hiçbir ClusterIP tahsis etmez ve bu nedenle "clusterIP: None" ile başlık servisi olarak ayarlanmıştır.

Aşağıda YAML dosyasının detayları:


**StatefulSet Tanımı:**
- `apiVersion` ve `kind`, StatefulSet kaynağını tanımlar. `apiVersion: apps/v1` ve `kind: StatefulSet` ile bir StatefulSet tanımlanır.
- StatefulSet adı "web" olarak belirtilmiştir.
- `serviceName` alanı "nginx" olarak ayarlanmıştır, bu da bu StatefulSet'in "nginx" adlı başlık servisini hedeflediği anlamına gelir.
- `replicas`, başlatılacak pod sayısını belirtir. StatefulSet, 2 replika ile yapılandırılmıştır.
- `volumeClaimTemplates`, StatefulSet pod'larının depolama alanı (volume) için yapılandırma şablonunu içerir. Bu örnekte, "www" adında bir depolama alanı oluşturur ve her pod için 1Gi depolama talep eder. StatefulSet, her pod için bir `www` adlı PVC (Persistent Volume Claim) şablonu tanımlar. Bu PVC'ler `ReadWriteOnce` erişim moduna ve 1Gi depolama talebine sahiptir.
- `selector`, pod'ların bu StatefulSet'e nasıl dahil edileceğini belirler. Pod'lar, "app: nginx" etiketi ile seçilir.
- `template` içinde pod konfigürasyonu tanımlanır. Bu podlar StatefulSet tarafından yaratılır.
  - `containers`, bu pod içinde çalışacak konteynerleri tanımlar.
    - `image`, kullanılacak konteyner görüntüsünü belirtir. Pod şablonu, "app: nginx" etiketi ve bir "nginx" konteyneri içerir. Bu konteyner, "registry.k8s.io/nginx-slim:0.8" imajından çalışır.
    - `ports`, bu pod'un hangi portları dinleyeceğini belirtir. 80 numaralı bir portu açar.
    - Ayrıca, pod'ların `/usr/share/nginx/html` dizinine bir `www` adlı veri sürücüsünü bağlamak için bir `volumeMounts` tanımı vardır.

Bu StatefulSet, "nginx" başlık servisini hedef alır ve her pod'un kendine özgü DNS girdisine sahip olmasını sağlar. StatefulSet, bu pod'ları başlatmak, durdurmak ve yönetmek için kullanılır ve her pod'un veri depolama taleplerini karşılamak için PVC'leri tanımlar. Bu şekilde, ölçeklenebilir ve yönetilebilir bir web sunucusu uygulaması oluşturmak için kullanılabilir.

Bu basit StatefulSet örneği, özellikle veri tabanları veya diğer stateful iş yükleri için kullanılır ve her pod, benzersiz bir ağ kimliği ve isme sahiptir, bu da onları birbirinden ayırır ve her birinin kendi durumunu sürdürebilmesini sağlar.
