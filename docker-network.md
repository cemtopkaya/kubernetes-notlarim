# Docker Ağ Nasıl Çalışır

Elimde bir ubuntu 16.04 WSL ve içinde çalışan docker hizmetim var. 
Aşağıdaki iptables komutunun yardım çıkları biraz daha aşağıda oluşturulan kuralları anlamamızı sağlayacak:
```bash
--policy  -P chain target              Change policy on chain to target
--append  -A chain                     Append to chain
--new     -N chain                     Create a new user-defined chain
--jump -j target                       target for rule (kural için yapılacak eylem DROP/ACCEPT/REJECT gibi)
[!] --out-interface -o output name[+]  network interface name ([+] for wildcard)
[!] --in-interface -i input name[+]    network interface name ([+] for wildcard)
--match       -m match                 extended match (may load extension)
```

Bir konteyner çalışmıyor, bir ağ yaratılmamış ve `iptables -S` çıktısı şöyle:
```shell
[14:05:54] cemt:cem.topkaya $ sudo iptables -S
-P INPUT ACCEPT
-P FORWARD DROP
-P OUTPUT ACCEPT
-N DOCKER
-N DOCKER-ISOLATION-STAGE-1
-N DOCKER-ISOLATION-STAGE-2
-N DOCKER-USER
-A FORWARD -j DOCKER-USER
-A FORWARD -j DOCKER-ISOLATION-STAGE-1
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2
-A DOCKER-ISOLATION-STAGE-1 -j RETURN
-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP
-A DOCKER-ISOLATION-STAGE-2 -j RETURN
-A DOCKER-USER -j RETURN
```

Bu `sudo iptables -S` komutunun çıktısı, iptables kurallarını gösterir ve Docker ile ilgili özelleştirilmiş kuralları içerir. Aşağıda bu çıktıdaki önemli bölümlerin açıklamalarını bulabilirsiniz:

1. `-P INPUT ACCEPT`: Bu satır, "INPUT" zincirinin varsayılan politikasını belirtir. "ACCEPT" olarak ayarlandığı için, gelen trafik varsayılan olarak kabul edilir.
2. `-P FORWARD DROP`: Bu satır, "FORWARD" zincirinin varsayılan politikasını belirtir. "DROP" olarak ayarlandığı için, "FORWARD" zincirine gelen trafik varsayılan olarak reddedilir. Bu, host makinesinden geçip başka bir ağ arabirimine yönlendirilen trafik için önemlidir.
3. `-P OUTPUT ACCEPT`: Bu satır, "OUTPUT" zincirinin varsayılan politikasını belirtir. "ACCEPT" olarak ayarlandığı için, host makinesinden giden trafik varsayılan olarak kabul edilir.
4. Bu satırlar, özel iptables zincirlerini tanımlar. Docker ve Docker izolasyonu için kullanılan özel zincirlerdir.
```shell
-N DOCKER
-N DOCKER-ISOLATION-STAGE-1
-N DOCKER-ISOLATION-STAGE-2
-N DOCKER-USER
```
5. `-A FORWARD -j DOCKER-USER`: Bu satır, "FORWARD" zincirine gelen trafik için "DOCKER-USER" zincirine yönlendirme yapar.
6. `-A FORWARD -j DOCKER-ISOLATION-STAGE-1`: Bu satır, "FORWARD" zincirine gelen trafik için "DOCKER-ISOLATION-STAGE-1" zincirine yönlendirme yapar.
7. `-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT`: Bu satır, Docker köprü ağı üzerinden geçen trafik için tanımlanmış bir kuraldır. Bu kural, ilişkili veya kurulmuş bağlantıları kabul eder.
8. `-A FORWARD -o docker0 -j DOCKER`: Bu satır, Docker köprü ağına yönlendirilen trafik için "DOCKER" zincirine yönlendirme yapar.
9. `-A FORWARD -i docker0 ! -o docker0 -j ACCEPT`: Bu satır, Docker köprü ağından gelen ve Docker köprü ağına gitmeyen trafik için kabul edilme kuralıdır.
10. `-A FORWARD -i docker0 -o docker0 -j ACCEPT`: Bu satır, Docker köprü ağından gelen ve Docker köprü ağına yönlendirilen trafik için kabul edilme kuralıdır.
11. `-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2`: Bu satır, Docker izolasyonunun ilk aşamasını temsil eder. Docker konteynerleri arasındaki iletişimi sınırlayan bir kuraldır. İlk aşamadan geçmeyen trafik, "DOCKER-ISOLATION-STAGE-2" zincirine yönlendirilir.
12. `-A DOCKER-ISOLATION-STAGE-1 -j RETURN`: Bu satır, "DOCKER-ISOLATION-STAGE-1" zincirine gelen trafik için bir "RETURN" kuralıdır.
13. `-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP`: Bu satır, Docker izolasyonunun ikinci aşamasını temsil eder. Bu kural, Docker köprü ağına yönlendirilen trafik için bir düşürme (DROP) kuralıdır.
14. `-A DOCKER-ISOLATION-STAGE-2 -j RETURN`: Bu satır, "DOCKER-ISOLATION-STAGE-2" zincirine gelen trafik için bir "RETURN" kuralıdır.
15. `-A DOCKER-USER -j RETURN`: Bu satır, "DOCKER-USER" zincirine gelen trafik için bir "RETURN" kuralıdır.

Bu iptables kuralları, Docker'ın ağ yönlendirmesi ve izolasyonunu sağlamak için kullanılan özelleştirilmiş kuralları içerir. Bu kurallar, gelen ve giden trafik yönlendirmesi, Docker konteynerler arası izolasyon ve ağ güvenliği için tasarlanmıştır.

![image](https://github.com/cemtopkaya/kubernetes-notlarim/assets/261946/882a8ccd-c374-4804-8b53-fe92f6c9676f)

Bu çıktı, Docker'ın ağ yönlendirmesi ve izolasyonu için kullanılan iptables kurallarını gösteriyor. İşte bu çıktıdaki önemli bölümlerin açıklamaları:

1. `-P INPUT ACCEPT`, `-P FORWARD ACCEPT`, `-P OUTPUT ACCEPT`: Bu satırlar, varsayılan iptables politikalarını belirtir (`-P` anahtarı "policy" anlamında). "ACCEPT" olarak ayarlandığı için gelen, yönlendirilen ve çıkan trafik varsayılan olarak kabul edilir.
2. `-N DOCKER`, `-N DOCKER-ISOLATION-STAGE-1`, `-N DOCKER-ISOLATION-STAGE-2`, `-N DOCKER-USER`: Bu satırlar, **özel iptables zincirlerini (chains) tanımlar** (`-N` anahtarı "new" anlamında). Docker, bu zincirleri kullanarak ağ yönlendirmesi ve izolasyonu yönetir.
3. `-A FORWARD -j DOCKER-USER`, `-A FORWARD -j DOCKER-ISOLATION-STAGE-1`: Bu satırlar, "FORWARD" zincirine yönlendirmeler ekler (`-A` anahtarı "append" anlamında). İlk olarak, gelen trafik "DOCKER-USER" zincirine yönlendirilir ve ardından "DOCKER-ISOLATION-STAGE-1" zincirine yönlendirilir. Bu, Docker'ın özelleştirdiği izolasyon katmanlarına giden trafik için bir başlangıç noktasıdır.
4. Aşağıdaki satırlar, Docker köprü ağı (**docker0**) üzerinden geçen trafik için yönlendirme kurallarını belirtir. Özellikle, gelen trafik ile ilişkili(**RELATED**) veya kurulmuş bağlantıları (**ESTABLISHED**) kabul eder (ACCEPT), ardından bu trafik Docker konteynerlerine yönlendirilir.
```
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER`, `-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
```

   ![docker0 Ağ arayüzüne](https://github.com/cemtopkaya/kubernetes-notlarim/assets/261946/307fe62f-ab9d-47d9-83a3-9ccc93473397)

5. `-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2`: Bu satır, Docker izolasyonunun ilk aşamasını temsil eder. Docker konteynerleri arasındaki iletişimi sınırlayan bir kuraldır. İlk aşamadan geçmeyen trafik, "DOCKER-ISOLATION-STAGE-2" zincirine yönlendirilir.
6. `-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP`: Bu satır, Docker izolasyonunun ikinci aşamasını temsil eder. Bu kural, Docker köprü ağına yönlendirilen trafik için bir düşürme (DROP) kuralıdır. Bu, konteynerler arasında iletişimi kısıtlar.
7. `-A DOCKER-USER -j RETURN`: Bu satır, "DOCKER-USER" zincirine gelen trafik için bir "RETURN" kuralıdır. Bu, kullanıcı tarafından özelleştirilebilecek bir zincirdir.

Bu kurallar, Docker'ın ağ yönlendirmesi ve izolasyonunu sağlamak için kullanılır. İptables kuralları, Docker'ın iç çalışma mantığını ve ağ yönlendirmesinin nasıl yapılandırıldığını açıkça gösterir.

## IPTables Zinciri Nedir?
"Iptables zinciri" terimi, iptables adlı Linux tabanlı bir güvenlik duvarı yönetim aracının temel yapı taşlarından biridir. 
Iptables, gelen ve giden ağ trafiğini kontrol etmek ve filtrelemek için kullanılır. 
Iptables, çeşitli "zincirler" (chains) içerir ve bu zincirler, paketlerin nasıl işleneceğini ve hangi kurallara tabi tutulacağını tanımlar.

Iptables zincirleri aşağıdaki gibi temel yapı taşlarıdır:

1. **INPUT Zinciri:** Gelen paketler bu zincire yönlendirilir. Bu zincirde tanımlanan kurallar, host makinenin kendisine yönlendirilen trafiği etkiler.
2. **OUTPUT Zinciri:** Host makinesinden çıkan paketler bu zincire yönlendirilir. Bu zincirde tanımlanan kurallar, host makinesinin dış dünyaya gönderdiği trafiği kontrol eder.
3. **FORWARD Zinciri:** Paketler, host makinesinden geçip başka bir ağ arayüzüne yönlendirildiğinde bu zincire yönlendirilir. Bu zincirde tanımlanan kurallar, paketlerin başka bir hedefe yönlendirilmesini veya reddedilmesini sağlar.

Iptables zincirleri, bu temel üçünün yanı sıra özel zincirler de içerebilir. Örneğin, Docker gibi uygulamalar özelleştirilmiş zincirler oluşturabilir ve kendi ağ yönlendirme kurallarını uygulayabilir.

Her iptables zinciri, bir dizi kurallardan oluşur. Bu kurallar, paketlerin ne yapılacağını belirler. Kurallar, genellikle aşağıdaki üç kısımdan oluşur:

1. **Filtreleme (Filter):** Paketin kabul edilip edilmeyeceğini belirler. Örneğin, kaynaktan gelen paketleri kabul etmek veya reddetmek için kullanılır.
2. **NAT (Network Address Translation):** Ağ adresi çevirisi yapar. Örneğin, iç ağdaki özel IP adreslerini dış dünyada kullanılabilir bir IP adresine çevirir.
3. **Mangle (Manipulation):** Paketin başlık bilgilerini değiştirmek için kullanılır. Örneğin, hız sınırlama veya QoS (Kalite Hizmeti) uygulamaları için kullanılabilir.

Her zincirdeki kurallar sırasıyla işlenir ve bir kural eşleşirse, paket üzerinde tanımlanan işlemler uygulanır. Iptables zincirleri, ağ trafiğini yönlendirmek, filtrelemek ve güvenlik duvarı işlevi sağlamak için kullanılır. Bu kurallar, Linux işletim sistemi altında çalışan sistem yöneticileri ve ağ yöneticileri tarafından yapılandırılır.

### Örnek Kurallar

`iptables -A INPUT -p icmp -j DROP` komutu, IP tablolarında bir kural ekler. 

Bu komutun her bir bölümünün anlamı şu şekildedir:

- `iptables`: IP tabloları aracını çağırır.
- `-A INPUT`: INPUT zincirine bir kural ekler. Bu, gelen bağlantıları kontrol eder.
- `-p icmp`: ICMP protokolünü belirtir. ICMP, Internet Control Message Protocol'ün kısaltmasıdır ve genellikle hata mesajlarını iletmek veya ağ cihazlarına sorgu göndermek için kullanılır.
- `-j DROP`: Eğer bir paket bu kurala uyuyorsa, DROP işlemi gerçekleştirilir. DROP, paketin hedefe ulaşmasını engeller ve istemciye bir geri dönüt mesajı göndermez.

Bu durumda, `-j` seçeneği "jump" (atla) anlamına gelir ve belirli bir işlemi belirtir. 
Bu komut, ICMP protokolünü kullanan tüm gelen paketlerin düşürülmesini sağlar. 
Yani, sunucu ICMP ping isteklerini yanıtlamaz hale gelir. 
Bu, genellikle güvenlik nedenleriyle yapılır, çünkü saldırganlar ping isteklerini sunucunun varlığını tespit etmek için kullanabilir.

