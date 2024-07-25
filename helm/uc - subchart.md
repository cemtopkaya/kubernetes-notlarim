# Ana ve Yavru Chartlar

Şimdi, grafiklerin nasıl değiştirileceğini öğrendik ve değerleri nasıl geçersiz kılacağımızı anlıyoruz; ancak bir ana grafik, `charts/` dizininde bulunan bir alt grafik içerebilir. 
Bu alt grafiklerin her birinin kendi değer dosyası vardır. Yani alt grafik, ana grafiğin bir çocuğudur ve o `charts` dizinindedir. 

### Alt Grafiklerin Özellikleri

Alt grafik, bağımsız bir grafik olduğundan, `charts/` dizininin üstündeki hiçbir şeye erişemez. Bu nedenle, alt grafiğin `values.yaml` dosyasında tanımlanan herhangi bir şey, kendi seviyesinin üstündeki değerlere erişemez. Ana grafiğin alt grafiğin değerlerini açıkça geçersiz kılabileceğinden bahsettiğimde, bu ana grafiğin `values.yaml` dosyasıdır. Ana grafik, alt grafik değerine açıkça atıfta bulunursa, alt grafikte bulunan değeri geçersiz kılabilir. 

![image](https://github.com/user-attachments/assets/72a4f995-3edd-4655-aed3-f1c7d5a65f15)

Bu, bağımlılıklarınız için değerleri ana grafikte, yani ana grafik içinde tanımlamanıza olanak tanır; böylece alt grafiği değiştirmek zorunda kalmazsınız. Eğer mevcutsa, bir global değer hem ana grafik hem de alt grafik tarafından erişilebilir. Bu değerler, `global` anahtar kelimesi kullanılarak açıkça tanımlanmalıdır.

![image](https://github.com/user-attachments/assets/526344c0-2a5e-4508-a920-31e3286f6d1e)

![image](https://github.com/user-attachments/assets/b8f5d4b3-620e-4d06-87e2-d54aeb4ea045)



### Alt Grafiğe Atıfta Bulunma

Alt grafiğe atıfta bulunmak için, ana grafik içinde `somechart.mappingvalues.firstOne` şeklinde referans vereceğiz. Daha sonra alt grafiğe gittiğimizde, `subchartValue.somechartvalue` şeklinde bir değer göreceğiz. Ardından bir global değere atıfta bulunacağız. Bu, değerlerimizi tanımlama şeklimiz açısından önemlidir.

### Uygulamalı Örnek

Şimdi, bunu gerçek bir grafikte nasıl göründüğünü görelim. Kubernetes kümemizde Helm yüklü. Bu, daha önce kullandığımız aynı küme. Burada indirilmiş bir WordPress grafiğim var ve bu grafiği genişletmeye gideceğim. Z anahtarını kullanarak genişleteceğim. Detaylı bilgiye ihtiyacım yok. Grafiği genişletelim. 

WordPress dizinini göreceğiz. Eğer `helm show values` komutunu `./wordpress` üzerinde çalıştırırsam, birçok bilgi alacağız. Burada önemli olan, bu WordPress kurulumu için bir bağımlılık olan bir MariaDB grafiğinin bulunması. 

### MariaDB Grafiği

Eğer yukarı kaydırırsak, veritabanıyla ilgili bazı şeyleri görmeye başlayacağız. Devam edersek, bu dış veritabanı olacak, yani `.Values.externalDatabase`, bu ana grafiği referans alır. Yukarı çıktığımızda, MariaDB grafik yapılandırmasını göreceğiz. Bu, `.Values.mariadb` ve buradaki her şeydir. Örneğin, `MariaDB enabled true` ve `rep- .replication.enabled false` gibi değerler, gerçek MariaDB grafiği için geçersiz kılmalardır.

WordPress grafiğine girdiğimizde, `charts` klasörünü göreceğiz. `charts/` dizininde listeleme yaptığımızda, MariaDB grafiğini göreceğiz. MariaDB'ye girdiğimizde, `values.yaml`, `chart.yaml` gibi dosyaları içeren tam bir grafik olduğunu göreceğiz. Üst düzeyde tanımlanan değerler, o belirli MariaDB grafiğindeki değerleri geçersiz kılacaktır. Bu, ana grafiğinize bağımlılık olarak kullanılan alt grafikleriniz olduğunda akılda tutulması gereken bir noktadır.

### MariaDB Değer Dosyası

MariaDB kendi başına bağımsız bir grafik olduğundan, MariaDB grafiğinde bulunan değerlere bakmak isterseniz, `helm show values` komutunu kullanabilirsiniz. WordPress dizinindeyiz, bu yüzden `./charts/mariadb/` dizinine gideceğim ve burada çok fazla değer olduğu için çıktıyı `less` ile yönlendireceğim. Gördüğünüz gibi, bu MariaDB grafiğinde bulunan değer dosyasıdır ve WordPress grafiğinden tamamen ayrıdır. Bu değerler, özellikle MariaDB örneği için geçerlidir.

Bu değerleri, doğru sözdizimi ile alt grafik adını ve ardından anahtar ve değeri kullanarak erişebilirsiniz. Bu, bağımlılıklarınızı ana grafiklerinizle dağıtırken kontrol etmenizi sağlar. 

### Sonuç

Bu ders için bu kadar. Bu bölümde grafiklerin nasıl değiştirileceğini gördük. Grafiklerin dilini anlıyoruz. Şimdi, bu derste alt grafiklerle ve grafiklerinizin bağımlılıklarıyla nasıl çalışacağımızı konuştuk; böylece bunların ortamınıza uygun bir şekilde yapılandırıldığından emin olduk. Bir sonraki bölümde, grafiklerde daha ileri tekniklere, örneğin kancalar (hooks) ve testler gibi konulara gireceğiz. Benim adım Mike, bir sonraki derste görüşmek üzere!
