---
layout: post
title: Security Onion Lab Ortamı Kurulumu
categories: zararli-yazilim some wiki
---

![logo](/../post_images/onion/auf.jpg)

(Aufhocker alman mitolojisi bulunan bir yaratıktır. Ormanda yalnız gezen kurbanlarının üzerine saldırdırlar.) / https://tr.pinterest.com/pin/423619908680945983/


* TOC
{:toc}


## Security Onion Nedir?
---

![logo](/../post_images/onion/sec.png)

Security Onion, saldırı tespiti, enterprise security monitoring ve log yönetimi için ücretsiz ve açık kaynaklı bir Linux dağıtımıdır. 

Elasticsearch, Logstash, Kibana, Snort, Suricata, Bro, Wazuh, Sguil, Squert, CyberChef, NetworkMiner ve diğer birçok güvenlik araçlarını içermektedir.





Bu dokümantasyonda Security Onion linux dağıtımını ve içerisindeki araçları nasıl kullanacağımıza dair detaylı açıklamalarda bulunacağım. Bu dokümantasyonu oluştururken https://securityonion.net adresinden ve kendi tecrübelerimden paylaşacağım.


### Network Security Monitoring nedir?
---

Ağ Güvenliği İzleme ağınızdan siber saldırıları algılama ve analiz yapmak için veri toplanmak demektir.


Security Onion üç temel işlevi bir araya getirmektedir:

- Full Packet Capture(ağdaki tüm paketleri kaydetme)
- Network ve Host Based Instrusion Detection System (hids ve nids)
- Güçlü ve işe yarar analiz araçları


#### Network IDS Nedir?
---
Ağ tabanlı izinsiz giriş algılama sistemleri/ saldırı tespit sistemi (NIDS), sırasıyla ağ trafiğinianaliz eder ve algılanan olaylar için log ve uyarı verilerini sağlar.  Security Onion üzerinde NIDS olarak, BRO, SNORT ve Suricata yazılımları bulunmaktadır.

#### Host IDS Nedir?
---
Security Onion, Host tabanlı izinsiz giriş tespiti/saldırı tespit için, (Windows, Linux ve Mac OS X) için ücretsiz, açık kaynaklı bir yazılım olan Wazuh ile birlikte gelmektedir. Wazuh agent'larını ağınızdaki uç noktalara kurduğunuzda bu uç noktalarda neler olup bittiğine dair görünürlük elde edilmektedir. 

Wazuh, log analizi, dosya bütünlüğü kontrolü(file integrity check), policy monitoring, rootkit tespiti, gerçek zamanlı uyarı ve active response gerçekleştirebilmektedir. Bir analist olarak, host tabanlı olayları ağ tabanlı olaylarla ilişkilendirebilmek, başarılı bir saldırıyı tespit etmekte çok önemlidir.

#### Analiz Araçları?
---
Full Packet Capture ve IDS verileriyle, analistin parmak uçlarında oldukça güzel miktarda veri oluşmaktadır. Security Onion içerisinde bu verileri analiz etmek için bazı araçlar bulunmaktadır.

##### Sguil Nedir?
---
Sguil, “Ağ Güvenliği İzlemesi için Analist Konsolu” diyebileceğimiz bir yazılımdır. Sguil Snort, Suricata ve Wazuh alarmlarını görüntülemek için tek bir GUI sağlar. Daha da önemlisi Sguil, doğrudan sisteminizde oluşmuş bir alarmdan paket yakalamaya (Wireshark/NetworkMiner aracılığıyla) veya alarmı tetikleyen tam oturumunu görmenizi sağlamaktadır.

##### Squert:
---
Squert, Sguil için bir web uygulaması arayüzüdür.

##### Elasticsearch , Logstash ve Kibana nedir?
---
Kısaca artık bu üçlüye ElasticStack denilmektedir. Security Onion içerisinde bulunan Elasticsearch nosql bir veritabanıdır. Logstash elasticsearch veritabanına bir çok farklı yoldan(tcp, udp, file, jdbc...) gibi yöntemlerle veri aktarılmasını sağlayan yazılımdır.

Kibana ise elasticsearch veritabanı üzerinde bulunan verilerin bir dashboard'ta analizlerini, görselleştirmelerini yapabileceğiniz bir yazılımdır. Bu üç yazılım security onion üzerinde kullanılarak, elde edilen tüm verilerin(ids, hids ve diğer araçlar) analiste uygun bir şekilde sunarak çözüm sağlamaktadır.


## Security Onion Mimarisi
---
Aşağıda Security Onion ve Elastik Stack için mevcut mimariyi ve dağıtım senaryolarını gösteren birkaç diyagram bulunmaktadır.

(https://securityonion.readthedocs.io/en/latest/architecture.html#deployment-types)

![so](/../post_images/onion/deployment1.png)

Daha Detaylı bir Diagram

![so](/../post_images/onion/deployment2.png)

### Security Onion Dağıtım Mimarileri
---
Security Onion, dağıtımı client/server modelinini kullanmaktaydı fakat elasticstack'in dahil edilmesiyle birlikte node/cluster yapısına geçiş yapıldı.

Bu şu anlama gelmektedir. Siz bir master sunucu olan security onion dağıtımı kurdunuz. Daha sonra forward node olarak isimlendirilen sadece veri ileten node kurulumları yapabilirsiniz(daha önce bunlara sensör deniyordu) Bir veya birden fazla depolama node'u kullanabilirsiniz. Bunlara daha detaylı değineceğiz.


![so](/../post_images/onion/arch1.jpg)


#### Dağıtık Kurulum Mimarisi
---
- Önerilen dağıtım türüdür
- Bir master sunucudan, bir veya daha fazla forward düğümden(node) ve bir veya daha fazla depolama düğümünden(node) oluşmaktadır.

![so](/../post_images/onion/arch1.png)


#### Heavy Mimari
---
- Yalnızca standart bir dağıtılmış dağıtım mümkün değilse önerilir.
- Bir master sunucudan ve bir veya daha heavy düğümden oluşur.


![so](/../post_images/onion/arch2.png)

#### Standalone Mimari
---
- Ana sunucu bileşenlerini, sensörü ve Elastic Stack bileşenlerini çalıştıran tek bir sunucudan oluşur.


![so](/../post_images/onion/arch3.png)


#### Sonuç Olarak
---
Security onion, network ve uçnokta güvenliği için ücretsiz ve açık kaynak olarak piyasaya sürülen bu yazılımları birbirine entegre ederek hızlı bir kurulum sağlıyor. Bu kurulumları da farklı mimari seçenekleri sunarak, ister bir lab ortamında, ister kendi şirketinizde, ister kurumsal bir ağda kullanıp çözüm üretebilirsiniz.


### Donanım Gereksinimleri
---
Security Onion, Elasticstack ve NIDS gibi yazılımlar bulundurmaktadır. Bu yazılımların kaynak ihtiyaçlarını sizin ne amaçla kullanacağıza bağlı olarak değişebilir. Ben bu yazımda bir LAB ortamına göre düşünmekteyim, fakat Security Onion bu konuda şu şekilde tavsiye vermektedir.

https://securityonion.readthedocs.io/en/latest/hardware.html


## Security Onion Kurulumu
---
Security Onion kurulumu, standart bir ubuntu kurulumundan farklı değildir. İlk kurulum sırasında herhangi bir fark yoktur, bugüne kadar başka bir linux dağıtımı bile kuran kişilerin kolayca kurabileceği bir dağıtımdır.

Security onion kurulumu tamamlandığında, masaüstüne "Setup" altına bir kısayol oluşturur. Burası tam olarak security onion'ı nasıl kuracağınıza bağlı olarak değişen bir dizi konfigurasyon girmenizi istemektedir. Dilerseniz bu kısımda ne gibi değişiklikler yapabiliriz bunlardan bahsedelim.


#### Ubuntu üzerine Security Onion Kurulumu
---
Eğer daha önce elinizde kurulu olan bir ubuntu dağıtımı varsa bunun üzerine security onion kurulumu yapabilirsiniz. Bu adımlar için aşağıda verdiğim linki takip edebilirsiniz.

https://securityonion.readthedocs.io/en/latest/installing-on-ubuntu.html


### İndirme ve İSO İşlemleri
---
Security Onion'u kurmak için, Security Onion ISO imajını indirebilir veya standart bir Ubuntu 16.04 ISO indirebilir ve üzerinde Security Onion PPA ve paketlerimizi ekleyerek kurlum sağlayabilirsiniz. PPA ve paketleri sadece Ubuntu 16.04 ile uyumludur.


İndirme adresi :
https://github.com/Security-Onion-Solutions/security-onion/blob/master/Verify_ISO.md

HER ZAMAN indirilen ISO dosyasını doğrulayın! 
Doğrulamayı nasıl yapacağınızı bilmiyorsanız, şu talimatları izleyin: https://github.com/Security-Onion-Solutions/security-onion/blob/master/Verify_ISO.md


### Standart Kurulumdan Sonra
---

Şimdi klasik bir linux dağıtımı gibi kurulumu tamamladınız ve masaüstüne geldiniz. İlk yapmanız gereken şey eğer herhangi bir sanallaştırma ortamında bulunuyorsanız Vmware guest tools'u kurmanızı tavsiye ederim. Çünkü diğer türlü masaüstü çözünürlüğü, host ve guest arasında dosya kopyalama gibi işlemleri tam olarak gerçekleştiremiyorsunuz.

Guest araçlarını kurmak için Vmware ya da Virtualbox ortamı kullanıyorsanız öncellikle guest tools cd'sini sanal sunucuya import etmeniz gerekiyor. Daha sonra bu dosyaları sistemde herhangi bir yere kopyalayıp kurulum dosyasını çalıştırınız.


### Setup ile Başlıyoruz...
---

İlk olarak setup'a tıkladıktan sonra bir hoşgeldin mesajı ile karşılaşıyoruz.

![so](/../post_images/onion/ss1.png)

Dilerseniz komut satırından 

```
sudo sosetup
```

Komutuyla çalıştırabilirsiniz.


Setup daha sonra size ağ konfigurasyonunun nasıl olacağını soracaktır. Burada DHCP ya da statik IP tanımlarınızı yaptıktan sonra reboot olacaktır.

![so](/../post_images/onion/ss2.png)

Bu adımı da geçtikten sonra size "Production Mode" ya da "Evaluation Mode" mu kullanmak istediğinizi soracaktır. 

Evaluation mode bazı tanımların önceden yapıldığı, lab ortamı için uygun olan seçenektir. Fakat daha özel ayarlar yapmak istiyorsanız Production mode'u seçin ve devam edin.


Yeni bir master node mu eklemek istiyorsunuz yoksa var olan bir master node var mı diye soruyor. Yeni diyoruz.

![so](/../post_images/onion/ss3.png)


Yeni bir user ve parola soracaktır. Bu user kibana içinde kullanacağınız user olacaktır.

![so](/../post_images/onion/ss4.png)



Burada SecurityOnion'ın sizin için karar vermesini ve önceden tanımlı ayarları kullanmasını istiyorsanız best practicles'ı seçin. (ilk defa kuranlar için önerilir)

![so](/../post_images/onion/ss5.png)

Burada IDS için hangi kural setini kullanacağımızı bize sormaktadır.  Ben uzun süredir takip ettiğim Emerging Threats Open Kurallarını seçeceğim.


![so](/../post_images/onion/ss6.png)


Şimdi IDS olarak Snort ya da Suricata seçmemizi bekliyor, ben daha önceki yazılarımda da kullandığım Suricata'yı seçeceğim.

![so](/../post_images/onion/ss7.png)

Network servislerini aktifleştiriyoruz.

![so](/../post_images/onion/ss8.png)

Daha sonraki konfigurasyonlara dokunmadan tamamlayabilirsiniz. Kurulumu ve konfigurasyonları tamamladıktan sonra masaüstüne Kibana, Sguel ve Squert için kısayolları oluşturmuş olduğunu göreceksiniz.


Kurulumu bu şekilde tamamlamış olduk eğer burada merak ettiğiniz başka konu varsa şuradan detaylı bilgiye erişebilirsiniz.

https://securityonion.readthedocs.io/en/latest/production-deployment.html#install


Kibana arayüzüne ulaşmak için masaüstündeki kibana kısayoluna tıkladıktan sonra dashboard menüsüne erişebilirsiniz.


Eğer hiç alarm oluşmadığını görüyorsanız, aşağıdaki komutu çalıştırarak Security Onion üzerindeki örnek pcap'leri tcpreplay ile yeniden oynatabilir ve alarm oluşmasını sağlayabilirsiniz.

```
tcpreplay -i ens33 /opt/samples/*.pcap
```


Bir sonraki yazımda Security Onion üzerinde detaylı trafik analizinin nasıl yapılacağını aktarmaya çalışacağım, okuduğunuz için teşekkürler.

Samet Sazak
smtszk@gmail.com

---







