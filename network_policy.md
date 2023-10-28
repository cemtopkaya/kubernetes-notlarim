# Network Policy

Kubernetes içinde "Network Policy" (Ağ Politikası), bir Kubernetes küme içindeki pod'ların ağ trafiğini nasıl kontrol edeceğinizi ve sınırlayabileceğinizi belirlemenize yardımcı olan bir kaynaktır. 
Network Policy, ağ güvenliği, izolasyon ve erişim kontrolünü geliştirmek için kullanılır ve birçok senaryoda faydalıdır.

İşte Network Policy'nin ne işe yaradığının bazı temel nedenleri:

1. **Ağ Güvenliği:** Network Policy, kötü niyetli veya istenmeyen ağ trafiğini engellemek için kullanılabilir. Özellikle çoklu kullanıcıların veya uygulamaların bulunduğu bir Kubernetes kümesinde, güvenlik açıklarını en aza indirmek için ağ trafiğini izole etmek önemlidir.
2. **İzolasyon:** Network Policy, farklı projeler, bölümler veya uygulamalar arasında pod izolasyonunu sağlar. Bu, bir pod'un diğer podlara zarar vermesini veya izinsiz erişim sağlamasını önler.
3. **Erişim Kontrolü:** Network Policy, belirli pod'lara erişimi sınırlamak veya belirli kaynaklara (örneğin veritabanlarına veya API'ye) erişimi kontrol etmek için kullanılabilir. Örneğin, yalnızca belirli pod'ların belirli bir veritabanına erişmesine izin verilebilir.
4. **Veri Koruma:** Network Policy, hassas verileri taşıyan pod'ların sadece yetkilendirilmiş pod'lara veya kullanıcılara erişmesine izin vererek veri güvenliğini artırabilir.
5. **Uygulama Performansı:** Network Policy, ağ trafiğini optimize etmek ve uygulama performansını artırmak için kullanılabilir. İstenmeyen trafiği filtrelemek veya yüksek erişim gereksinimlerine sahip pod'ların performansını optimize etmek mümkün olabilir.

Network Policy, Kubernetes'teki birçok ağ uygulamasında kullanışlıdır ve ağ konfigürasyonunu daha kesin ve güvenli bir şekilde yapılandırmak için kullanılabilir. Ancak, doğru bir şekilde yapılandırılması ve yönetilmesi gereken karmaşık bir özelliktir ve Kubernetes ağ politikalarını oluştururken dikkatli olunmalıdır.

![image](https://github.com/cemtopkaya/kubernetes-notlarim/assets/261946/621e71ad-115c-4fb7-8166-d0faf21dcf28)

İşte basit bir Network Policy YAML örneği:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
    - Ingress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            role: frontend
```

Bu Network Policy, "allow-nginx" adını taşır ve "nginx" adındaki pod'lara gelen ağ trafiğini denetler. Bu politika yalnızca belirli bir etikete (örneğin, "role: frontend") sahip pod'lardan gelen trafiği kabul eder.

Aşağıda bu YAML dosyasının detayları:

- `apiVersion` ve `kind`: Network Policy kaynağının türünü ve sürümünü belirtir.
- `metadata`: Network Policy'nin meta verilerini içerir, bu örnekte politikanın adını belirtir.
- `spec`: Network Policy'nin ayrıntılarını içerir. `podSelector`, politikanın uygulanacağı pod'ları seçer; bu örnekte "app: nginx" etiketine sahip pod'lar seçilmiştir.
- `policyTypes`: Hangi tür ağ politikasının tanımlandığını belirtir; bu örnekte sadece "Ingress" (giriş) trafiği tanımlanmıştır.
- `ingress`: Gelen trafiği nasıl kontrol edeceğinizi belirtir. Bu örnekte, yalnızca "role: frontend" etiketine sahip pod'lardan gelen trafiğin kabul edileceği belirtilmiştir.

Bu, çok basit bir Network Policy örneğidir. Gerçek dünya senaryolarında, daha karmaşık politikalar oluşturmanız gerekebilir, ancak bu örnek, temel kullanımı anlamanıza yardımcı olacaktır.
