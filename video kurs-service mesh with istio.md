"Service Mesh with Istio" Video eğitiminden aldığım notları burada yazacağım.

### Şimdiden Kaynaklar
- https://learn.microsoft.com/tr-tr/azure/aks/istio-about
- [Neden docker'dan kubernetes'e geçiyoruz](https://youtu.be/H1d8Wwx4vkI?si=_BpjhgoZhZEhSIRz&t=258)
- [Neden istio'ya ihtiyacımız var?](https://youtu.be/H1d8Wwx4vkI?si=lk4rNRhsmBnP1usk&t=390)
- [Traffic mirroring, also called shadowing](https://youtu.be/H1d8Wwx4vkI?si=xRg7CkVrqm8u-4DK&t=1482)

# Service Mesh ne yapar?

![image](https://github.com/user-attachments/assets/8e9eda73-932d-4886-bd61-328719655a22)

Bir servis mesh'i (service mesh), mikro hizmetlerin birbirleriyle olan iletişimlerini yönetmek ve optimize etmek için kullanılan bir katmandır. Istio gibi bir servis mesh ile aşağıdaki önemli işlevleri gerçekleştirebilirsiniz:

1. **Servis Keşfi (Service Discovery):**
   - Istio, hizmetlerin birbirlerini bulmasını sağlar. Örneğin, bir mikro hizmetin başka bir mikro hizmete ulaşabilmesi için adresini bilmesi gerekmez; Istio bu adreslemeyi otomatik olarak yönetir.

2. **Yük Dengeleme (Load Balancing):**
   - Istio, trafiği birden fazla konteyner arasında dengeler. Bu, kaynakların daha verimli kullanılmasını ve hizmetlerin daha erişilebilir olmasını sağlar. Örneğin, beş ön plan konteyneri, iki arka plan konteyneri ve iki veri konteyneri arasında yük dengelemesi yapılabilir.

3. **Hata Kurtarma (Failure Recovery):**
   - Bir konteynerin arızalanması durumunda, Istio trafiği otomatik olarak başka bir konteynere yönlendirir. Bu, hizmetin kesintisiz çalışmasını sağlar.

4. **Metrik Sağlama (Metrics Provisioning):**
   - Istio, hizmetlerin ne kadar kullanıldığını gösteren metrikler sağlar. Bu veriler, hizmetlerin performansını izlemek ve iyileştirmek için kullanılabilir.

5. **Kota ve İstek Sınırlaması (Quota and Rate Limiting):**
   - Istio, belirli hizmetlerin belirli miktarlarda kullanılmasını sağlayarak aşırı yüklenmeyi önler. Örneğin, bir istemci çok fazla istek yaparsa, bu istekler sınırlanabilir.

6. **Denetim İzleri (Audit Trails):**
   - Giriş ve çıkış trafiği için denetim izleri sağlar. Bu, güvenlik olaylarını ve performans sorunlarını izlemek için kullanılabilir.

7. **İletişim Güvenliği (Communication Security):**
   - Mikro hizmetler arasındaki tüm iletişimleri şifreleyerek güvenli hale getirir. Bu, veri güvenliğini artırır ve yetkisiz erişimi önler.

8. **Kimlik Doğrulama ve Yetkilendirme (Authentication and Authorization):**
   - Istio, kimlik tabanlı kimlik doğrulama ve yetkilendirme sağlar. Örneğin, bir ön plan konteyneri sadece belirli arka plan konteynerlerinden gelen istekleri kabul edecek şekilde yapılandırılabilir.

9. **Trafik Yönetimi (Traffic Management):**
   - Istio, trafik yönetimi ile Canary testi gibi ileri seviye trafik yönlendirme stratejileri sağlar. Örneğin, yeni bir hizmet versiyonuna sadece %10 oranında trafik yönlendirebilir veya kullanıcıların coğrafi konumlarına göre farklı versiyonlara yönlendirme yapabilirsiniz.

10. **Loglama ve İzleme (Logging and Monitoring):**
    - Istio, hizmet mesh'in içindeki olayları izlemek ve loglamak için güçlü araçlar sunar. Grafana gibi kontrol panelleri ile hizmetlerin performansını izleyebilir ve sorunları tespit edebilirsiniz.

11. **Hata Testi (Fault Injection):**
    - Sisteme hata enjekte ederek, sistemin bu hatalara nasıl tepki verdiğini ve iyileştiğini görebilirsiniz.

12. **Altyapıdan Soyutlama (Infrastructure Abstraction):**
    - Istio, konteynerleri alttaki donanım ve altyapıdan soyutlar. Bu, hizmetlerin daha taşınabilir ve yönetilebilir olmasını sağlar.

Istio gibi bir servis mesh, mikro hizmetlerin yönetimini kolaylaştırır, güvenliği artırır ve uygulamaların performansını optimize eder. Bu özellikler, büyük ve karmaşık dağıtımların daha verimli ve güvenilir bir şekilde çalışmasını sağlar.

# İstio Bileşenleri

Şimdi Istio'yu oluşturan bileşenlerden bahsedeceğiz. Sol tarafta, Kubernetes master içinde bulunuyoruz ve `kubectl get pods -n istio-system` komutunu kullanarak Istio için çalışan tüm podları görebiliyoruz. Sağ tarafta ise Istio mimarisinin bir diyagramı var.

![image](https://github.com/user-attachments/assets/e7700af4-6721-4b59-9f0b-d942aaa53e85)

![image](https://github.com/user-attachments/assets/5d6bd3c7-819e-4c9a-8f50-86e304daa0a0)


### Istio Mimarisi ve Bileşenleri

**1. Proxy (Envoy)**
   - **Görevleri:**
       - Dinamik servis keşfi,
       - yük dengeleme,
       - TLS sonlandırma,
       - sağlık kontrolleri (health check),
       - aşamalı dağıtımlar (staged rollouts),
       - hata enjeksiyonu (fault injection) vb.
   - **Katman:** OSI modelinin 7. katmanı (uygulama katmanı).
   - **Özellikleri:** Envoy, her podun içine yerleştirilen bir sidecar proxy olarak çalışır. Tüm pod trafiği bu proxy üzerinden geçer, bu da uygulama kodunda değişiklik yapmadan Istio'nun devreye girmesini sağlar. Gelen istekleri inceleyebilir ve içeriklerine göre kararlar alabilir.

**2. [Pilot](https://youtu.be/H1d8Wwx4vkI?si=QmKO2Z-IduZOZiJk&t=1713)**
   - **Görevleri:** Proxy'lere konfigürasyon gönderir, servis keşfi sağlar, akıllı yönlendirme ve dayanıklılık sağlar. Istio Pilot, Istio Service Mesh içinde trafik yönetimi, güvenlik, politika yönetimi ve gözlemlenebilirlik gibi önemli işlevleri yerine getirerek, mikroservis mimarilerinin yönetimini kolaylaştırır. Bu, geliştiricilerin uygulamalarını daha güvenli ve verimli bir şekilde yönetmelerine olanak tanır.
   - **Özellikleri:** Pilot, sağlık kontrolleri ve kullanılabilir podlar hakkında bilgi alarak, istekleri uygun podlara yönlendirir. Bu, uygulamanın daha verimli çalışmasını sağlar.
   - Istio Pilot, temel olarak aşağıdaki işlevleri yerine getirir:
       - **Trafik Yönetimi: ** Pilot, mikroservisler arasındaki trafiği yönetmek için yapılandırmalar oluşturur. Bu, yük dengeleme, yönlendirme ve hata toleransı gibi özellikleri içerir. Pilot, uygulama ortamını modelleyerek, bu model üzerinden yapılandırmaları Envoy proxy'lerine iletir.
       - **Güvenlik ve Politika Yönetimi:** Pilot, mikroservisler arasındaki iletişimi güvenli hale getirmek için TLS şifrelemesi ve kimlik doğrulama gibi güvenlik özelliklerini yönetir. Ayrıca, servislerin birbirleriyle nasıl iletişim kuracağını belirleyen politikaları da uygular.
       - **Gözlemlenebilirlik:** Pilot, sistemin gözlemlenebilirliğini artırmak için telemetry (izleme) verilerini toplar ve bu verileri analiz eder. Bu sayede, uygulama performansı ve güvenliği hakkında bilgi sahibi olunmasını sağlar.
       - **Yapılandırma Yönetimi:** Pilot, Istio yapılandırmalarını Galley'den alarak, bu bilgileri bir araya getirir ve servis kayıt defterlerinden (örneğin Kubernetes API sunucusu) elde edilen bilgilerle birleştirir. Bu süreç, veri düzlemi yapılandırmalarını oluşturur ve bunları bağlı olan Envoy proxy'lerine iletir.

**3. Citadel**
   - **Görevleri:** Kullanıcı kimlik doğrulaması (authentication) ve yetkilendirmesi (authorization), kimlik bilgisi yönetimi (credential management), sertifika yönetimi (certificate management) ve trafik şifreleme (traffic encyrption).
   - **Özellikleri:** Citadel, kimlik doğrulama ve yetkilendirme için gereken kimlik bilgilerini ve sertifikaları yönetir. İlgili kimlik bilgilerini doğrulamak için Citadel'e başvurulur.

**4. Mixer**
   - **Görevleri:** Erişim kontrolü, kullanım politikaları ve telemetri verilerinin yönetimi.
   - **Özellikleri:** Kullanıcı yetkilendirmesi ve erişim politikalarının yanı sıra, telemetri verilerini toplayarak Prometheus ve Grafana gibi sistemlerle entegrasyon sağlar. Mixer, Datadog ve AWS gibi popüler sistemlerle entegrasyon için birçok eklentiye sahiptir.

**5. Galley**
   - **Görevleri:** Altyapı API'si için arayüz sağlar.
   - **Özellikleri:** Politika ayarları gibi konularda Istio'nun diğer bölümlerine gerekli bilgileri iletir. Mikro hizmetlerin API çağrılarını yorumlar ve bu çağrıları ilgili hizmetlere yönlendirir.

**6. Telemetri ve Politika Podları**
   - **Görevleri:** Telemetri verilerini toplar ve politikaları uygular.
   - **Özellikleri:** Bu iki işlev, Mixer tarafından yönetilse de farklı podlarda çalışır.

### Diğer Bileşenler

- **Grafana:** Istio'nun performans ve durumunu izlemek için kullanılan bir kontrol panelidir.
- **Prometheus:** Telemetri verilerini toplayıp saklar ve Grafana ile entegre çalışarak verileri görselleştirir.

### Özet

Istio'nun kontrol düzlemi, Citadel, Mixer ve Pilot'tan oluşur ve bu bileşenler proxy ile etkileşim halindedir. Bu mimari, mikro hizmetlerin yönetimini ve güvenliğini sağlar, hizmet keşfi, yük dengeleme, kimlik doğrulama ve yetkilendirme gibi kritik işlevleri yerine getirir. Galley, API çağrılarını yorumlar ve ilgili hizmetlere ileterek bu süreci destekler. Grafana ve Prometheus ise izleme ve telemetri işlevlerini gerçekleştirir.

Bu derste, Istio'yu oluşturan bileşenlerin genel bir özetini sunduk. Daha sonraki derslerde, bu bileşenlerin nasıl çalıştığını ve nasıl entegre olduklarını daha detaylı inceleyeceğiz.
