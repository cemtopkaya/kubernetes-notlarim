# emptyDir ile Konteynerler Arası Paylaşılan POD'a Özgü Dizin

Bu volume'ların kullanım amaçları:

```yaml
- emptyDir: {}
  name: empty-dir
```
**empty-dir**: Geçici dosyalar veya konteynerler arası veri paylaşımı için kullanılabilir. Geçici veri depolamak için kullanılan boş bir dizindir. Pod başlatıldığında oluşturulur ve pod silindiğinde yok edilir. Aynı pod içindeki konteynerler arasında veri paylaşımı için kullanışlıdır.

```yaml
- emptyDir:
    medium: Memory
  name: dshm
```
**dshm**: Bu da bir EmptyDir volume'dur, ancak medium: Memory belirtildiği için RAM'de oluşturulur. Bu, çok hızlı okuma/yazma gerektiren geçici veriler için kullanılır. Genellikle /dev/shm olarak monte edilir ve shared memory için kullanılır. PostgreSQL'in performans optimizasyonları için kullanılabilir. Örneğin, geçici tablolar veya indeksler için hızlı erişim sağlar.


Aşağıdaki POD içinde iki konteyner ayaklanır. Birisi kendi içindeki `/shared-data/data.txt`dosyasına yazarken diğeri aynı dizini paylaşarak yazılan satırları `tail -f /shared-data/data.txt` komutuyla ekrana basar.

```yaml
cem.topkaya@cem:~/empty-dir$ cat writer-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: writer-pod
spec:
  containers:
  - name: reader-container
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do tail -f /shared-data/data.txt; sleep 10; done"]
    volumeMounts:
    - name: shared-volume
      mountPath: /shared-data
  - name: writer-container
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date) >> /shared-data/data.txt; sleep 5; done"]
    volumeMounts:
    - name: shared-volume
      mountPath: /shared-data
  volumes:
  - name: shared-volume
    emptyDir: {}
```


