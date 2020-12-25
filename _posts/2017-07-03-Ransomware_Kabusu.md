---
layout: post
title: Ransomware Kabusu
categories: zararli-yazilim wiki
---

## Ransomware Kâbusu


![ransomware_kabusu-16](/../post_images/ransomware_kabusu/Ransomware_Kabusu-16.jpg)


Son zamanlarda şirketlere karşı yapılan siber saldırılar sonucunda birçok şirkette hatta ülkelerde siber farkındalık önem kazanarak artmaya başladı. Bununla beraber şirketlerde SOC (Security Operation Center) ekipleri hızla kurularak sistemler ekipler tarafından 7/24 incelenmeye başlanıldı.
Bu yazımızda SOC çalışanlarının karşılarına çıkabilecek alert durumları için senaryolar oluşturarak pratik yapacağız.

Bir şirkette SOC çalışanı olarak öğleden sonraki vardıyanız için şirkete geldiniz. Ofisinize doğru ilerlerken yan ofisinizden biri hızla ve telaşlı olarak odadan çıkıyor ve yüzyüze geliyorsunuz.
-	Odadan çıkan kişi sizi görünce heyecanla "siz SOC ekibindensiniz değil mi?"
-	Sen "Evet, bir sorun mu var?" demeye kalmadan seni ofisine çekiyor.
-	Bilgisayarının başına geçirerek "bilgisayarımdaki bütün dosyalar şifrelenmiş ve açılması için para ödemem gerektiği yazıyor."
Görünüşe göre  klasik bir ransomware saldırısı son zamanlarda tün dünyanın sıkıntısı.
-	Nasıl bir davranış gösteren ransomware bilmediğinden olayı öncelikle endişeli çalışana bilgisayarı şirketin networkünden çıkarmasını istiyorsun.
-	Endişeli çalışan "Ne olmuş bilgisayarıma" diye merakla soruyor.
-	Sen "Ransomware saldırısı" diye yanıtlıyorsun ama karşındakinin boş bakışlarından birşey anlamadığı kesin.
Şirket içerisinde yapılan bilgi güvenliği farkındalık eğitimlerini düşünerek iç geçiriyorsun. İnsan faktörü her zaman önde oluyor.
-	Dosyalarını kurtarabilmek için backup'ı olup olmadığını soruyorsun.
-	Endişeli çalışan "backup??"
-	Şifrelenen dosyaları için çok fazla şansının olmadığını söylüyorsun.
-	Endişeli çalışan artık üzgün ve sinirli olarak "Bu neden benim başıma geldi? Kim bunu bana yaptı?"
-	Çalışana bakarak SOC sistemleri içerisinde network trafiği dinlediğinizden dolayı süpheli trafik için kesin alarm oluşturduğunu ve bunu tespit edebileceğini söylüyorsun.
-	Endişeli çalışan size bakarak "Nasıl olduğunu öğrenmek istiyorum" diyor.

Ransomware  saldırısının kimin yaptığını söyleyemezsiniz ama bilgisayarı nasıl bulaştığını bulabilirsiniz. Sisteminizdeki network alarmlarına baktığınızda ransomware aktivitesinin olduğu tek bir IP adresi görüyoruz. Etkilenen bilgisayarın IP adresine ait tüm alertleri ve o zamana ait network trafiğini inceliyoruz.

**Aşağıda etkilenen bilgisayara ait network trafiği dosyaları bulunmaktadır.**

Etkilenen bilgisayara ait PCAP dosyası : [2016-10-15-traffic-analysis-exercise.pcap.zip](http://www.malware-traffic-analysis.net/2016/10/15/2016-10-15-traffic-analysis-exercise.pcap.zip) (Zip parolası : infected)

Network trafiğinin oluşturduğu alarmları içeren zip arşivi : [ 2016-10-15-traffic-analysis-exercise-alerts.zip](http://www.malware-traffic-analysis.net/2016/10/15/2016-10-15-traffic-analysis-exercise-alerts.zip) (Zip parolası : infected)

### Göreviniz:

Elimizde alertler ve network trafiği var. Elimizdeki bilgiler ile bilgisayarı etkilenmiş çalışanın ne olduğunu göstericek bir rapor yazalım. Rapor içeriğinde
- Ransomware aktivitesinin günü ve zamanı
- Çalışanın bilgisayarına ne olduğunun özeti bulunmalıdır.

## Çözüm

Bu noktadan itibaren elimizdeki dosyaları inceleyerek network trafik analizine başlayacağız. Çalışmanın verimli geçebilmesi için bu kısmı okumadan önce kendinizin yukarıdaki network trafiğine ait dosyaları incelemenizi tavsiye ediyoruz.

İyi bir araştırmacı öncelikle elimizdeki alarmlara bakarız. Etkilenen bilgisayarın IP adresine (10.14.106.192) göre network trafiği elimizde ve oluşan alarmlara göre bilgisayarın nasıl etkilendiğini buradan bulabiliriz.

İlk olarak alarmların bulunduğu txt dosyasının açarak incelemeye başlıyoruz. Alarmların oluştuğu zamana göre ilk görevimiz olan "Ransomware aktivitesinin günü ve zamanı" tespit edebiliriz.

Tetiklenen alarmların isminden tanıdık isim olan **Rig EK** gözümüze çarpıyor. Rig zararlı yazılımların site içerisinde ansızın yönlendirilen bir url içerisinden bulaşmalarını sağlamaktadırlar. Burada da "**10.14.106.192**" IP adresli host bilgisayarımızın "**50.56.223.21**" adresline **80** portunu kullanarak bağlandığını görüyoruz.

```
Count:5 Event#8.1634 2016-10-14 22:14:42 UTC
ET CURRENT_EVENTS Evil Redirector Leading to EK Jul 12 2016
50.56.223.21 -> 10.14.106.192
IPVer=4 hlen=5 tos=0 dlen=1361 ID=0 flags=0 offset=0 ttl=0 chksum=12172
Protocol: 6 sport=80 -> dport=49450

Seq=0 Ack=0 Off=5 Res=0 Flags=******** Win=0 urp=35823 chksum=0
------------------------------------------------------------------------
Count:1 Event#8.1639 2016-10-14 22:14:45 UTC
ET CURRENT_EVENTS RIG Landing URI Struct March 20 2015
10.14.106.192 -> 109.234.36.251
IPVer=4 hlen=5 tos=0 dlen=503 ID=0 flags=0 offset=0 ttl=0 chksum=45390
Protocol: 6 sport=49467 -> dport=80

Seq=0 Ack=0 Off=5 Res=0 Flags=******** Win=0 urp=4549 chksum=0
------------------------------------------------------------------------
Count:7 Event#8.1640 2016-10-14 22:14:46 UTC
ET CURRENT_EVENTS RIG EK Landing Sep 12 2016 T2
109.234.36.251 -> 10.14.106.192
IPVer=4 hlen=5 tos=0 dlen=1361 ID=0 flags=0 offset=0 ttl=0 chksum=44532
Protocol: 6 sport=80 -> dport=49467

Seq=0 Ack=0 Off=5 Res=0 Flags=******** Win=0 urp=56031 chksum=0
------------------------------------------------------------------------
Count:1 Event#8.1647 2016-10-14 22:14:46 UTC
ET CURRENT_EVENTS RIG Exploit URI Struct March 20 2015
10.14.106.192 -> 109.234.36.251
IPVer=4 hlen=5 tos=0 dlen=698 ID=0 flags=0 offset=0 ttl=0 chksum=45195
Protocol: 6 sport=49467 -> dport=80

Seq=0 Ack=0 Off=5 Res=0 Flags=******** Win=0 urp=15636 chksum=0
------------------------------------------------------------------------
Count:1 Event#8.1652 2016-10-14 22:14:49 UTC
ET CURRENT_EVENTS RIG Payload URI Struct March 20 2015
10.14.106.192 -> 109.234.36.251
IPVer=4 hlen=5 tos=0 dlen=522 ID=0 flags=0 offset=0 ttl=0 chksum=45371
Protocol: 6 sport=49471 -> dport=80

Seq=0 Ack=0 Off=5 Res=0 Flags=******** Win=0 urp=31621 chksum=0
------------------------------------------------------------------------
Count:10 Event#8.1653 2016-10-14 22:14:50 UTC
ETPRO CURRENT_EVENTS RIG/Sundown/Xer EK Payload Jul 06 2016 M2
109.234.36.251 -> 10.14.106.192
IPVer=4 hlen=5 tos=0 dlen=1361 ID=0 flags=0 offset=0 ttl=0 chksum=44532
Protocol: 6 sport=80 -> dport=49471

Seq=0 Ack=0 Off=5 Res=0 Flags=******** Win=0 urp=43975 chksum=0
------------------------------------------------------------------------
Count:1 Event#8.1665 2016-10-14 22:15:22 UTC
ETPRO TROJAN Ransomware/Cerber Checkin 2
10.14.106.192 -> 31.184.234.0
IPVer=4 hlen=5 tos=0 dlen=37 ID=7987 flags=0 offset=0 ttl=127 chksum=40462
Protocol: 17 sport=62374 -> dport=6892

len=17 chksum=15442
------------------------------------------------------------------------
Count:11 Event#8.1669 2016-10-14 22:14:46 UTC
ETPRO CURRENT_EVENTS RIG EK Flash Exploit Mar 29 2016
109.234.36.251 -> 10.14.106.192
IPVer=4 hlen=5 tos=0 dlen=1361 ID=0 flags=0 offset=0 ttl=0 chksum=44532
Protocol: 6 sport=80 -> dport=49467

Seq=0 Ack=0 Off=5 Res=0 Flags=******** Win=0 urp=12111 chksum=0
------------------------------------------------------------------------
Count:1 Event#8.1693 2016-10-14 22:17:03 UTC
ET CNC Ransomware Tracker Reported CnC Server group 19
10.14.106.192 -> 173.254.231.111
IPVer=4 hlen=5 tos=0 dlen=52 ID=11479 flags=2 offset=0 ttl=127 chksum=50352
Protocol: 6 sport=49520 -> dport=80

Seq=2228766067 Ack=0 Off=8 Res=0 Flags=******S* Win=8192 urp=47303 chksum=0

```

Host bilgiyara yapılan saldırı konusunda daha detaylı bilgi alabilmek için web adresi hakkında ve siteye kurulan bağlantı hakkında detaylı bilgiye ihtiyacımız var. Alarmdan yola çıkılarak elde edilen bilgiler ile pcap dosyası wireshark kullanılarak detaylı olarak incelenebilir. Veya sistemlerde kullanılan IDS monitoring toolları ile zaten bu bilgiye alarmlar ile direk ulaşılabilir. Öncelikle SOC merkezinde SELKS kullanıldığını düşünür isek SELKS içerisinde **evebox** kullanarak istenilen detaylı alarmı gözlemleyebiliriz.

![ransomware_kabusu-09](/../post_images/ransomware_kabusu/Ransomware_Kabusu-09.png)

İlk alarm "**ET CURRENT_EVENTS Evil Redirector Leading to EK Jul 12 2016
50.56.223.21 -> 10.14.106.192**" ve **80** portunu kullandığını görüyoruz.

![ransomware_kabusu-02](/../post_images/ransomware_kabusu/Ransomware_Kabusu-02.png)
![ransomware_kabusu-03](/../post_images/ransomware_kabusu/Ransomware_Kabusu-03.png)

Alarm detaylı incelendiğinde **50.56.223.21** IP adresi "**unwrappable.com**" web sitesine ait olduğunu görüyoruz. Ne yazıkki payload içerisinde gözlemlendiği gibi burada web sitesi için içerik gzip ile sıkıştırılmış burada daha fazla inceleme yapamayacağız. HTML'i daha iyi inceleyebilmek için isteği export etmemiz gerekmektedir.

İkinci alarmı incelediğimizde "RIG Landing URI Struct" olayı daha da anlamlandırabiliyoruz.
![ransomware_kabusu-08](/../post_images/ransomware_kabusu/Ransomware_Kabusu-08.png)

Host bilgisayarımız "unwrappable.com" adresine bağlandıktan sonra "rew.kaghaan.com" adresine yönlendirildiğini görüyoruz.

"unwrappable.com" adresinden gönderilen HTML'i daha iyi inceleyebilmek için isteği export etmemiz gerekmektedir. Dosyayı export edebilmek için genellikle wireshark kullanılır. Wireshark ile export etmek rahat ve kolaydır.
Pcap dosyası wireshark ile açılır. Wireshark içerisinde **File -> Export Objects -> HTTP ** seçilerek pcap içerisinde dosyaların bulunduğu pencere açılır.

![ransomware_kabusu-05](/../post_images/ransomware_kabusu/Ransomware_Kabusu-05.png)

HTTP dosyalarının bulunduğu liste içerisinden bizim istediğimiz "**unwrappable.com**" web adresine ait html dosyasını bilgisayarımıza kaydederiz.

![ransomware_kabusu-06](/../post_images/ransomware_kabusu/Ransomware_Kabusu-06.png)

HTML kodu incelediğimizde kod içerisine enjecte edilen script "body" etiketinden hemen sonra olduğunu görüyoruz. Zaten oluşan 2.alarm ile de bu adrese bağlandığını tespit etmiştik. 
![ransomware_kabusu-07](/../post_images/ransomware_kabusu/Ransomware_Kabusu-07.png)

Dördüncü alarm "RIG Exploit URI Struct" incelendiğinde host bilgisayar artık zararlı site olan rew.kaghaan adresinden "application/x-shockwave-flash" GET isteğinden gözlemlediğimiz kadarıyla zararlı flash dosyasını indirir.

![ransomware_kabusu-10](/../post_images/ransomware_kabusu/Ransomware_Kabusu-10.png)

Beşinci alarm "RIG Payload URI Struct" artık host bilgisayar zararlı binary dosyasını indirdiğini görüyoruz.

![ransomware_kabusu-11](/../post_images/ransomware_kabusu/Ransomware_Kabusu-11.png)

İndirilen binary dosya çalıştırıldığı zaman "Ransomware/Cerber Checkin 2" alarm oluşuyor. Bu da göteriyorki **Cerber Ransomware** iş başında. Cerber UDP kullanarak "31.184.234.0" IP adresine bağlanıyor.
![ransomware_kabusu-12](/../post_images/ransomware_kabusu/Ransomware_Kabusu-12.png)

Cerber Ransomware gördükten sonra oluşan bir diğer alarm "Cerber Bitcoin Address Check" içerisinde host bilgisayarın "btc.blockr.io" adresine bağlandığını görüyoruz. 

![ransomware_kabusu-13](/../post_images/ransomware_kabusu/Ransomware_Kabusu-13.png)

"btc.blockr.io" adresi host bilgisayar içerisinde dosyalar şifrelendikten sonra kullanıcının karşısına çıkan dosyaların geri açılabilmesi için yatırması gerektiği bitcoin miktarı ve diğer gereklilikler konusunda bilgi veren web sitesidir.

![ransomware_kabusu-14](/../post_images/ransomware_kabusu/Ransomware_Kabusu-14.png)

Oluşan bir diğer alarm "ffoqr3ug7m726zou.19jmfr.top" adresinde şüpheli bir network trafiği tespit ediyor. Bu adres ile ilgili diğer istekler detaylı incelendiğinde bu adresinde bir önceki adres gibi Cerber'in dosyaları şifreledikten sonra açtığı bilgilendirme web adresi olduğunu görüyoruz.

![ransomware_kabusu-15](/../post_images/ransomware_kabusu/Ransomware_Kabusu-15.png)

### Host Bilgisayara Yapılan Ransomware Saldırısının Özeti:

 **Ransomware aktivitesinin günü ve zamanı:**
 **2016-10-14 tarihinde yaklaşık olarak saat 22:14 UTC civarlarındadır.**

Elimizdeki pcap dosyasının ve alarmların incelenmesi sonucunda çalışanın bilgisayarına olanlar özetle, öncelikle kullanıcının bağlandığı web adresi içerisinde bulunan script kullanıcıyı başka bir siteye yönlendiriyor. Yönlendirilen site içerisinde Flash exploit ile zararlı binary dosyası host bilgisayara indiriliyor. Binary dosyası çalıştırılması ile kullanıcı "Cerber Ransomware"e maruz kalarak dosyaları şifreliyor.

Saldırı analizi orjinal çözüm PDF : [ 2016-10-15-traffic-analysis-exercise-answers.pdf.zip ](http://www.malware-traffic-analysis.net/2016/10/15/2016-10-15-traffic-analysis-exercise-answers.pdf.zip) (Zip parolası : infected)


## Son Notlar:
-------------------------------------------------------
Not: Bu yazıda kullanılan içerikler http://www.malware-traffic-analysis.net adresinden alınmış olup, yazarın kendisinden izin alınmış(@malware_traffic) ve düzenlenmiştir. Kendisine yardımları için teşekkür ederiz.

PS: All materials on this post are taken from http://www.malware-traffic-analysis.net. Special thanks to @malware_traffic for sharing.

