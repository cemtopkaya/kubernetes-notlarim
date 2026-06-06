# Job

Kubernetes’in “tek seferlik işler” için sunduğu bir nesne tipidir.

Özetle;
-   **Job** = Tek seferlik iş.
-   **Deployment** = Sürekli çalışan uygulama.
-   **CronJob** = Belirli zaman aralıklarında tekrar eden Job.


### Temel Mantık

-   **Job**, bir veya daha fazla Pod başlatır.
-   Pod’lar başarılı şekilde tamamlandığında Job da “tamamlandı” kabul edilir.
-   Tekrar eden, sürekli çalışan workload değildir; **Deployment** gibi sürekli ayakta kalmaz.
-   Genellikle batch işlemleri, script çalıştırma, veri işleme gibi görevler için kullanılır.

---

### Basit Örnek

Bir Job tanımı:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["echo", "Merhaba Kubernetes Job!"]
      restartPolicy: Never
```

Bu Job, bir Pod açar, `echo` komutunu çalıştırır ve iş bittiğinde Pod kapanır. Job da “tamamlandı” olarak işaretlenir.

---

### Kullanım Senaryoları

-   **Batch işlemleri**: Veri dönüştürme, rapor üretme.
-   **Tek seferlik script**: Migration, cleanup.
-   **Zamanlanmış işler**: Job’un zamanlanmış versiyonu CronJob ile yapılır.
