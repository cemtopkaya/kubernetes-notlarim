# Kubernetes’te Otomatik Ölçekleme Atölyesi

## Amaç
Bu atölye çalışmasında, Kubernetes'in en gücülü özelliklerinden biri olan otomatik ölçekleme (‘autoscaling’) mekanizmasını anlamayı ve kullanmayı öğreneceksiniz. Otomatik ölçekleme, kaynak kullanımındaki gerçek zamanlı değişikliklere yanıt olarak kaynak tahsisini artırır ya da azaltır. Bu özellik, sürekli dağıtım (‘continuous deployment’) için insan müdahalesini azaltarak sistem stabilitesini artırır.

### Bu Atölyede Neler Yapacaksınız?

- Kubernetes metrics API’sini kurma.
- Yatay Pod Otomatik Ölçekleyici (Horizontal Pod Autoscaler - HPA) oluşturma.
- Uygulamaya yükleme oluşturarak otomatik ölçekleyiciyi çalıştırma.

---

## Adımlar

### 1. Kubernetes Metrics API’sini Yükleme
Metrics API, Kubernetes’te otomatik ölçekleme mekanizmaları için gerekli olan kaynak kullanım verilerini sağlar.

#### Metrics API’yi Kurmak:
1. **Metrics Server deposunu klonlayın:**
   ```bash
   git clone https://github.com/kubernetes-incubator/metrics-server.git
   ```

2. **Standart konfigürasyonlarla Metrics API’yi kurun:**
   ```bash
   cd metrics-server/
   git checkout ed0663b3b4ddbfab5afea166dfd68c677930d22e
   kubectl create -f deploy/1.8+/
   ```

3. **Metrics Server podlarının durumunu kontrol edin:**
   ```bash
   kubectl get pods -n kube-system
   ```
   Podların durumu ‘Running’ olduğunda Metrics API başarıyla yüklenmiştir.

---

### 2. Yatay Pod Otomatik Ölçekleyiciyi (HPA) Oluşturmak
Bir HPA, belirli bir metrik (CPU, bellek vb.) kullanımına dayalı olarak pod replika sayısını artırır ya da azaltır.

#### HPA Konfigürasyonu:
1. **Uygulamanın YAML dosyasını düzenleyin:**
   [`train-schedule-kube.yml`](https://github.com/linuxacademy/cicd-pipeline-train-schedule-autoscaling/blob/example-solution/train-schedule-kube.yml) dosyasında gereken kod değişikliklerinin bir örneği için kaynak kod deposunun example-solution dalını kontrol edin.
   Kaynak kodun bir çatalını https://github.com/linuxacademy/cicd-pipeline-train-schedule-autoscaling adresinde oluşturun.
   - Aşağıdaki gibi bir CPU kaynak talebi (‘requests’) ekleyin:
     ```yaml
     resources:
       requests:
         cpu: 200m
     ```
   - YAML dosyasına Horizontal Pod Autoscaler tanımını ekleyin:
     ```yaml
     apiVersion: autoscaling/v2beta1
     kind: HorizontalPodAutoscaler
     metadata:
       name: train-schedule
       namespace: default
     spec:
       scaleTargetRef:
         apiVersion: apps/v1
         kind: Deployment
         name: train-schedule-deployment
       minReplicas: 1
       maxReplicas: 4
       metrics:
       - type: Resource
         resource:
           name: cpu
           targetAverageUtilization: 50
     ```

3. **YAML dosyasını Kubernetes’e uygulayın:**
   ```bash
   kubectl apply -f train-schedule-kube.yml
   ```

---

### 3. Yükleme Oluşturup HPA’yı Test Etme
HPA’nın çalışıp çalışmadığını görmek için uygulamada yapay yükleme (‘load generation’) oluşturun.

#### Yük Testi:
1. **Uygulamanın servis adını kontrol edin:**
   ```bash
   kubectl get svc
   ```
2. **Servis URL’ini kullanarak yükleme oluşturun:**
   Bir yük testi aracı (örn. Apache Benchmark veya k6) ile uygulamada CPU yükünü artırın:
   ```bash
   ab -n 10000 -c 100 http://<SERVIS_URL>
   ```
3. **Pod replika sayısındaki artışı izleyin:**
   ```bash
   kubectl get hpa
   kubectl get pods
   ```

---

## Sonuç
Bu atölyeyi tamamladığınızda, Kubernetes’te otomatik ölçekleme mekanizmasını kurma ve test etme konusunda temel bir bilgiye sahip olacaksınız. Gerçek zamanlı kaynak yönetimiyle sistem stabilitesini artırma yöntemlerini anlamış olacaksınız.

