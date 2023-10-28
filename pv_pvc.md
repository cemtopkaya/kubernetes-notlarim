# Persistent Volume (Kalıcı Depolama Birimi) ve Persistent Volume Claim (Kalıcı Depolama Talebi)

Persistent Volume (Kalıcı Depolama Birimi) ve Persistent Volume Claim (Kalıcı Depolama Talebi) kavramlarını çok basit bir örnekle inceleyelim:

Hayal edelim ki senin oyuncağını saklamak için özel bir kutu istiyorsun. Bu kutu, oyuncaklarını korumak için kullanabileceğin özel bir yerdir. İşte bu kutu, "Persistent Volume" olarak düşünülebilir.

- **Persistent Volume (PV - Kalıcı Depolama Birimi)**: PV, sanki oyuncaklarını saklamak için kullanabileceğin özel kutu gibidir. Özel kutun, oyuncaklarını korumak için kullanabileceğin bir depolama alanıdır. Her kutu farklı boyutlarda ve özelliklere sahip olabilir, örneğin büyük kutular, küçük kutular, su geçirmez kutular gibi. Her bir kutu, bir tür "PV" olur ve senin kullanımın için ayrılmıştır.

Şimdi, bu kutuları elde etmek için "Persistent Volume Claim" (Kalıcı Depolama Talebi) gereklidir.

- **Persistent Volume Claim (PVC - Kalıcı Depolama Talebi)**: PVC, sanki senin oyuncağını saklamak için özel bir kutu talep ettiğin bir not gibidir. PVC, bir tür talep veya istektir. "Ben bir büyük kutu istiyorum" ya da "Ben bir su geçirmez kutu istiyorum" gibi. PVC, senin ne tür bir depolama ihtiyacın olduğunu belirtir ve Kubernetes (veya sistem) bu talebe uygun bir PV (kutu) bulup sana tahsis eder.

Yani, PVC ile PV birleşerek senin için özel bir depolama alanı oluşturur. Senin oyuncağını bu depolama alanında saklayabilirsin. Ayrıca, bu depolama alanı senin istediğin gibi büyük veya küçük olabilir ve özel ihtiyaçlarını karşılayabilir.

Özetle, Persistent Volume, senin oyuncağını saklamak için kullanabileceğin özel bir kutu gibidir ve Persistent Volume Claim, bu özel kutuyu istediğin türdeki kutu için bir talep veya istek olarak düşünülebilir. 
Birlikte, senin için özel bir depolama alanı oluştururlar.

StatefulSet içindeki bir pod eşsiz bir isim ve ağ bilgisine sahiptir. Bu podlar kapanmışsa ve tekrar açılıyorsa aynı eşisiz isim, ağ ve kalıcı depolama alanlarına bağlanır.

Aşağıdaki resimde her bir pod için ayrı PVC'ler ve PV'leri göreceksiniz. Her tekrar başlatıldığını bu POD'lar aynı PV'lere bağlanır:

![image](https://github.com/cemtopkaya/kubernetes-notlarim/assets/261946/4f8fd288-abeb-42d6-8a84-b8505f44253e)
