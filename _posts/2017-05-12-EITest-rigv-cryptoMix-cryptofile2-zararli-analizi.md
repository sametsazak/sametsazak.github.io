---
layout: post
title: EITest-RIGV CryptoMix/Cryptofile2 Zararlı Analizi
categories: zararli-yazilim some
---

![2017-01-12-EITest-RIGV-01](/../post_images/2017-05-12-rigv-crytomix/logo.jpg)

("Griffon" veya "Griffin", genellikle aslan vücutlu, kartal kanatlı ve kafalı mitolojik yaratıktır.)

**Aşağıdaki dosyalar zararlı yazılım içermektedir. Tüm dosyaları sanal makine üzerinde çalıştırmanızı tavsiye ederiz, bu konuyla ilgili tüm sorumluluk kullanıcıya aittir.**

Zararlıya ait PCAP dosyası : [2017-05-09-EITest-Rig-V-sends-CryptoMix-ransomware.pcap.zip](http://www.malware-traffic-analysis.net/2017/01/12/2017-01-12-EITest-Rig-V-sends-CryptoMix-ransomware.pcap.zip) (Zip parolası : infected)

Tüm zararlı dosyaları içeren zip arşivi : [2017-05-09-EITest-Rig-V-sends-CryptoMix-malware-and-artifacts.zip](http://www.malware-traffic-analysis.net/2017/01/12/2017-01-12-EITest-Rig-V-sends-CryptoMix-malware-and-artifacts.zip) (Zip parolası : infected)

* TOC
{:toc}


## EITest Grubu ve RIG Zaralısı :

EITest topluluğu zararlı yazılım yaymak için RIG olarak isimlendirdikleri yazılımlar geliştirmişlerdir Bu grup kötü amaçlı yazılım dağıtmak için RIG EK kullanmaktadır. Zararlı yazılımların site içerisinde ansızın yönlendirilen bir url içerisinden bulaşmalarını sağlamaktadırlar. RIG'in bir kaç tane çeşiti bulunmaktadır;

**RIG-E :** Araştırmalara  göre EITEST genellikle Empire Pack denilen RIG EK çeşidini kullanmakta, topluluktaki pek çok kişi Empire Pack'i "RIG-E" olarak adlandırmaktadır. Kötü amaçlı yazılımı dağıtmak için EITEST tarafından yazılmış çoğu payload'ta RIG-E kullanıldığı gözlemlenmiştir.

**RIG-V :** RIG-E 2015 ten bu yana aynı url kalıplarını kullanırken, RIG-V'de biraz daha geliştirilerek farklı url kalıplarına geçirilmiş. Bu nedenle RIG-V'ye geliştirilmiş VIP versiyonuda diyebiliriz. Cerber veya CryptoMix/CryptFile2  gibi ransomware dağıtımları için RIG-V kullanılıyor.

Bu yazıda RIG-V ile bulaşan Cryptofile2 olarak da bilinen CryptoMix zararlısını inceleyeceğiz.

## Yayılma Şekli :

CryptoMix zararlısı EITest payload'u içeren beklenmedik bir kaç web sitesi tarafından yayılmaktadır. Zararlının bulunduğu web sitesini ziyaret etmeniz, EITest javascript payload'un çalışması için yeterlidir. RIG-V exploit kiti devreye girerek CryptoMix ransomware'ı ile bilgisayarınız şifrelenerek rehin alınmış olur.


![2017-01-12-EITest-RIGV-01](/../post_images/2017-05-12-rigv-crytomix/RigV-Crytomix-yayilma-sekli.png)

Son zamanlarda Activaclinics adlı web site içerisinde RIG-V javascript koduna rastlanmıştır. Bu web sitenin görünümü aşağıdaki gibidir.

![2017-01-12-EITest-RIGV-01](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix01.png)

## Saldırı Analizi:

Ransomware, web site içerisinde gezinirken hiç farkettirmeden devreye giren javascript kodu ile farklı yerlere  isteklerde bulunuyor ve bu işlemler sonucu bulaşmış oluyor. Trafik incelemesi esnasında bu yönlendirmeleri gözlemleyeceğiz. Fakat web sitesi dışında farklı yöntemlerle de bulaşabilir. Spam mail ile bu urllere yönlendirme işlemi gerçekleştirilebilir.

Activaclinics sayfasının kaynak kodunu incelediğimizde aşağıdaki payloadı görmekteyiz. Bu javascript kodunu incelediğimizde ise we.sedona.news domainine istekler yapıldığı görülebilir. Bu yönlendirilen url'den CryptoMix ransomware bilgisayarımıza istemeden de olsa indirilmiş olacaktır.

![2017-01-12-EITest-RIGV-06](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix06.png)

Zip dosyası içerisinde bulunan exe dosyası çalıştırıldığında ise ransomware tetiklenmiş olacaktır.

![2017-01-12-EITest-RIGV-02](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix02.png)

Yukarıdaki windows uyarısına OK cevabı verildiğinde crytomix zararlısı aktif hale gelir.

**Aktifleştiğinde;**

![2017-01-12-EITest-RIGV-04](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix04.png)

Resimde gördüğünüz gibi tüm dosyalarınızı şifreleyerek, sizden fidye karşılığında dosyalarınızı tekrar kazanabileceğinize dair bir mesaj iletiyor ve masaüstünde metin dosyasında nasıl yapacağınıza dair mesajlar bırakıyor.

Bahsedilen exe dosyasını Virustotal üzerinde sorguladığımızda aşağıdaki gibi bir sonuçla karşılaşıyoruz.

![2017-01-12-EITest-RIGV-03](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix03.png)

## Trafik Analizi

Zararlı yazılımın bulaşma anı ve o andaki trafiği Wireshark ile aşağıdaki görebiliriz. 

![2017-01-12-EITest-RIGV-05](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix05.png)

Yukarıda görüldüğü gibi Activaclinics sitesinin ziyaret edilmesinin ardından, site içerisinde bulunan javascript payloadının tetiklenmesi ile farklı bir IP adresine GET ve POST isteklerinde bulunulduğunu görebiliriz. Bu isteklerin we.sedona.news sitesine yapıldığı da görülmektedir. Bu url EITest grubunun CrptoMix zararlısını yaymak için kullandığı urllerden sadece bir tanesidir.

Bu işlemlerin gerçekleştirilmesi için site içerisinde yer alan javascript kodu tetiklenmektedir.

**Zararlı Yazılımın Bulaşması ile Oluşan Trafiğin Bro ile İncelenmesi**

    - mirivix@secops:~/CryptoMix$ bro -r 2017-05-09-EITest-Rig-V-sends-CryptoMix-ransomware.pcap.zip local

Yukarıdaki komut çalıştırıldığında; bro pcap içerisindeki kayıtlı trafiği kendi alanlarında sınıflandırarak log dosyalarına yazar. Oluşan log dosyaları aşağıdaki gibidir.

![2017-01-12-EITest-RIGV-07](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix07.png)

"conn.log" dosyası içerisindeki bağlantılar incelendiğinde öncelikle "81.177.139.122" IP adresine sonrasında ise "91.121.244.84" IP adresine bağlantıda bulunulduğu görülebilmektedir.

![2017-01-12-EITest-RIGV-08](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix08.png)

"http.log" dosyası içerisindeki istekler incelendiğinde ise activaclinics.com domaininde iken we.sedona.news domainine istekte bulunulduğu, sonrasında ise "91.121.244.84" adresine bağlanıldığı görülebilmektedir. Bu istekler Activaclinics sitesi içerisindeki javascript kodu ile gerçekleşmektedir.

![2017-01-12-EITest-RIGV-09](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix09.png)

**Domainler:**

    activaclinics.com - zararlı kodun bulunduğu site
    81.177.139.122 port 80 - we.sedona.news - Rig-V
    91.121.244.84 port 80 - 91.121.244.84 - CryptoMix bulaşması sonrası trafik

    supl@post.com - CryptoMix şifre çözme talimatları için ilk e-posta
    supl@oath.com - CryptoMix şifre çözme talimatları için ikinci e-posta


**Saldırı Tespit Sistemi Kullanarak Trafik Analizi**

Trafik analizi ve saldırı tespit sistemi ile zararlının tespiti için SELKS (Suricata/Elasticsearch/Logstash/Kibana/Scirius) sistemi kullanılabilir. SELKS içerisindeki Suricata ile pcap dosyası da kolaylıkla analiz edilebilir.

```
suricata -r /home/selks-user/2017-05-09-EITest-Rig-V-sends-CryptoMix-ransomware.pcap --runmode single -l /var/log/CryptoMix/
```

Yukarıdaki komut çalıştırıldıktan sonra pcap dosyası içerisinde bulunan trafik Suricata içerisinde bulunan kurallar ile analiz edilmiş olacaktır. Bu kurallar ile denetleme işlemi gerçekleştirildikten sonra oluşan alarmlar aşağıdaki gibidir.

![2017-01-12-EITest-RIGV-10](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix10.png)

Suricata pcap içerisindeki trafiğin zararlı bir trafik olduğunu tespit ettiği gibi RIG-V CrytoMix/Cryptofile2 zararlısı olduğunu da tespit ettiğini yukarıdaki alarmlarda görebiliriz.

**NOT:** Zararlı IP adresleri veya zararlılar sürekli güncellendiğinden SELKS içerisinde bulunan Suricata için yazılmış Scirius ile kuralları güncel tutmak gerekmektedir. Bir zararlı, kurallar güncellenmediğinden dolayı Suricata tarafından tespit edilemeyebilir. (Saldırı tespit edildikten sonra)


Oluşan alarmların kullanıcılar için görsel olarak incelenebilmesi için SELKS üzerindeki **Evebox** veya **Scirius** arayüzleri kullanılabilir. 

Kendi makinemiz üzerinde aynı trafiğin simüle edilmesi için Pcap dosyasını **"tcpreplay"** kullanarak oynatırız. Böylelikle Suricata tarafından oluşan alarmları daha anlaşılabilir şekilde inceleyebiliriz.

![2017-01-12-EITest-RIGV-11](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix11.png)

SELKS içerisinde bulunan Suricata arayüzünde alarmlar ve aktiviteleri aşağıdaki gibi gözlemlenebilir.

![2017-01-12-EITest-RIGV-12](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix12.png)

Evebox arayüzünde oluşan alarmlar aşağıdaki gibidir.

![2017-01-12-EITest-RIGV-13](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix13.png)

Evebox arayüzünde gözlemleyebileceğimiz eventler aşağıdaki gibidir.

![2017-01-12-EITest-RIGV-14](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix14.png)

Evebox arayüzünde oluşan alarm raporu ise aşağıdaki gibidir.

![2017-01-12-EITest-RIGV-15](/../post_images/2017-05-12-rigv-crytomix/rigv-cryptomix15.png)

## Son Notlar:
-------------------------------------------------------

Not: Bu yazıda kullanılan içerikler http://www.malware-traffic-analysis.net adresinden alınmış olup, yazarın kendisinden izin alınmış(@malware_traffic) ve düzenlenmiştir. Kendisine yardımları için teşekkür ederiz.

PS: All materials on this post are taken from http://www.malware-traffic-analysis.net. Special thanks to @malware_traffic for sharing.

## Referanslar:

- Logo - https://www.edupics.com/coloring-page-griffon-i11222.html
