---
layout: post
title: P2P W32.Sality Botnet Saldırı Analizi
categories: zararli-yazilim some
---

## Genel Bakış
--------------------------------------------

![botnet1](/../post_images/botnet/cokbasli.jpg) 

(Hidra, Yunan mitolojisinde anlatılan çok başlı bir yaratık.)

W32.Sality, ilk olarak 2003-2004 yılında görülmeye başlamış bir virüs olarak ortaya çıktı. Windows sistemleri etkileyen bu virüs, **"File infector"** olarak belirtildi. Kendisini başka dosyalara enjekte edebiliyor ve enjekte edilmiş dosyalar kullanıldığında sisteminize rahat bir şekilde bulaşmış oluyor ve aynı zamanda başka sistemlere de kendisini bulaştırma yeteneğine sahip oluyor. Bunun yanı sıra enfekte olmuş sistemler bir P2P(Peer-to-Peer) network ağına ortak olup, kendisini yaymak ve bot yöneticisi tarafından kontrol edilmesini sağlıyor. W32.Sality, genel bir bakış açısıyla aşağıdaki yeteneklere sahip:

  - Kendisini diğer dosyalara enjekte etmek ve çoğaltmak,
  - Spam Email göndermek,
  - Dağıtık yapıyı kullanarak parola kırmak(Brute-force),
  - Web sunucularına sızmak,
  - Klavye kayıtları(Keylogger),
  - DDOS saldırısı,
  - Bilgi Sızdırmak

**W32.Sality**, tüm bu yeteneklerin yanı sıra, kendisini kontrol eden kişi veya kişilerin daha fazla özellik eklemesine, çok hızlı bir şekilde yayılmasına ve tek bir merkezden kontrol edilmeyen bir p2p ağına sahip olması ile en tehlikeli zararlılardan birisi haline gelmiştir.

* TOC
{:toc}

## Klasik Tek Merkez C&C vs Sality P2P Botnet
------------------------------------------------------------
Klasik botnet yaklaşımında saldırgan, zombileştirdiği bilgisayarları tek bir merkezden yönetiyor. Bu bir IRC sunucusu, bir web paneli veya istemci-sunucu modeline dayalı herhangi bir yöntem olablir. Twitter üzerinden kontrol edilen ve android cihazları zombileştiren bir zararlı yazılım daha önce tespit edilmişti. [1] Bu şekilde bir botnetin kontrol edildiği sunucu ele geçirildiğinde ya da oyun dışı bırakıldığında büyük oranda tehditler engellenebilir. Fakat P2P botnetlerde durum bundan biraz farklıdır. Botlar birbiriyle bağlantı kurabilir, bot listelerini güncelleyebilir, iletişimini güçlendirebilir ve komuta merkezinin tek bir nokta olmadığı dağıtık bir yapı halinde kendisini sürdürmeye devam eder. Bot yöneticisi, diğer botların erişebildiği bir noktayı güncellediğinde, diğer botlar bundan haberdar olup, birbirlerini güncelleyeceklerdir.

Aynı zamanda bu iletişim, **"açık anahtarlı şifreleme"** yöntemi ile kullanılıyor. Tüm botlar public key(açık anahtar) ve bot yöneticisi private key(özel anahtar) sahip olduğundan iletişim tamamen şifrelenmiştir.

Genel bakışı 2 adet resim ile özetlediğimizde:

Command & Control Merkezli Yaklaşım:

![botnet1](/../post_images/botnet/1.png)

P2P Yaklaşımı:

![botnet](/../post_images/botnet/2.png)

[Kaynak](http://news.softpedia.com/news/Damballa-Enhances-Failsafe-as-P2P-Increasingly-Used-by-Malware-for-C-C-Communications-358583.shtml#sgal_0)

Sality botları davranışsal olarak aşağıdaki yetenekleri gösteriyor:
 - **A** botu diğer **B** botunun URL listesini **alabilir.**
 - **A** botu **B** botuna kendi URL listesini **gönderebilir.**
 - **A** botu **B** botuna bir diğer **C** botunun adresini(IP,Port) **gönderebilir.**

--------------------------------------------------------------------

**Peki, bu URL adreslerini ne yapıyor?**

Birazdan trafik incelemesini yaptığımızda göreceğiz, bu URL listelerini sürekli olarak kontrol ediyor. Kendisi için gerekli dosyaları bu şekilde indiriyor. Bu dosyaları kendi private(özel) anahtarı ile çözüyor ve kullanmaya başlıyor. Bot yöneticisi bu adresleri güncelleyerek, Sality botnete yön verebiliyor. Örneğin A web sitesine DDOS atılmasını istediğinde, sadece diğer botların erişebileceğini bir noktayı güncellemesi yeterli olacaktır. Peşinden gidebileceğiniz çok fazla nokta yok, zira hepsi aynı işi yapıyor olacaktır.


## Yayılma Şekli

**W32.Sality** kendisini dağıtmak için bir çok yöntem kullanmıştır. 2003'te ilk çıktığı andan itibaren kendisini bir çok dosyaya enjekte etmiş ve bu dosyalar paylaşıldıkça çoğalmıştır. Fakat Sality geliştikçe yaymak için kullanılan yöntemler de gelişmiştir. Büyük çoğunlukla spam email kampanyası yaparak hedeflere yönelmiştir. Bazı kumar oynatan web sitelerinin reklam propagandalarının içerisinde de bulunduğu gözlemlenmiştir.[4]

![botnet](/../post_images/botnet/casino.png)

[Kaynak](http://blog.checkpoint.com/2015/09/10/analysis-of-the-sality-gambling-campaign/)

## İstatistikler
-------------------------------------------------
2010 yılında Symantec firmasının yaptığı bir araştırmaya göre, Sality botnet olarak belirlenmiş IP adres istatistikleri aşağıdadır. **Türkiye** %4 ile hedef alınanlar listesindedir.

![botnet](/../post_images/botnet/istatistik.png)

[Kaynak:](https://www.symantec.com/connect/blogs/sality-botnet)


## Saldırı Senaryosu

[Czech Technical University ATG Group](http://agents.fel.cvut.cz/) tarafından başlatılmış olan ["The Malware Capture Facility Project"](http://mcfp.weebly.com/) 'ten alınmış W32.Sality enfeksiyonu yapılmış Windows sistemin yaklaşık **45** gün boyunca kaydedilen trafiğini analiz edeceğiz.

> **Aşağıdaki dosyalar zararlı yazılım içermektedir. Tüm dosyaları sanal makine üzerinde çalıştırmanızı tavsiye ederiz, bu konuyla ilgili tüm sorumluluk kullanıcıya aittir.**

[Zararlı Yazılım - 89828eec51d6fe22768c9364dcbb49b9.exe.zip](https://mcfp.felk.cvut.cz/publicDatasets/CTU-Malware-Capture-Botnet-66-1/89828eec51d6fe22768c9364dcbb49b9.exe.zip) (Zip parolası: infected)

[Pcap Dosyası - 2014-04-07_capture-win13.pcap](https://mcfp.felk.cvut.cz/publicDatasets/CTU-Malware-Capture-Botnet-66-1/2014-04-07_capture-win13.pcap)

## Saldırı Analizi
-------------------------------------------------------------
Analizime SecurityOnion sistemi üzerinde BRO IDS kullanarak başlıyorum. İlk olarak BRO loglarını inceleyeceğim. Daha sonra detaylı bir şekilde farkettiğim durumlara değineceğim. İndirmiş olduğumuz pcap dosyasını Bro'ya local konfigurasyon ile birlikte incelemesi aşağıdaki komutu kullanıyorum.

![botnet](/../post_images/botnet/bro1.png)

```
bro –r 2014-04-07_capture-win13.pcap local
```
Biraz uzun sürebilir bittiğinde aşağıdaki şekilde BRO loglarını aynı dizinde bulunmaktadır.

![botnet](/../post_images/botnet/bro2.png)

Daha sonra logları açıp incelemeden önce, SecurityOnion üzerinde kurulu olan ELSA ortamına aktararak, grafiksel olarak görüntüler elde etmek istiyorum. Bunun için BRO'nun analiz ettiği log dosyalarını **/nsm/bro/logs/current/** dizini altına kopyaladım.

```
https://localhost/elsa/ adresine giderek açabilirsiniz.
```

İlk olarak HTTP trafiği incelemek istiyorum. Top Client/Server IP'lerini listelediğimde "10.0.2.113" adresinin 19942 kere sayıldığını görüyorum. Bu enfekte edilmiş Windows host IP adresimiz. Diğer IP adresi benim lab ortamıma ait. Şimdilik görmezden gelebiliriz.

![botnet](/../post_images/botnet/bro3.png)

"Top Server IP" adreslerine baktığımda bazı adreslerin oldukça **fazla** bir trafik olduğunu görüyorum.

![botnet](/../post_images/botnet/bro4.png)

Şimdilik IP adreslerini bir kenera bırakıyorum ve en fazla trafiğin oldu siteleri inceliyorum.

![botnet](/../post_images/botnet/bro5.png)

Evet, hemen dikkatinizi çekti değil mi? Bazı adresler 700-800 defa ziyaret edilmiş. Grafikte anormalliğin olduğu bir kısım var.

![botnet](/../post_images/botnet/bro6.png)

Buraya tekrar döneceğiz, fakat şimdilik bir de diğer loglara bakalım. DNS logları ile devam ediyorum, herhangi bir DGA(Domain Generation Algorithm) kullanılıp kullanılmadağına bakacağım.

En fazla atılan DNS isteklerine baktığımda:

![botnet](/../post_images/botnet/bro7.png)

Hemen hemen benzeri bir tablo görüyorum. Bazı adreslere oldukça fazla, bazıları normal bir trafik gibi görünüyor. Listenin başında bulunan "msftncsi.com" Microsoft'a ait bir adres ve internet bağlantısının olup/olmadığını kontrol etmek için oluşturulmuş.[5] Bunun dışında herhangi bir DGA görüntüsü şimdilik görünmüyor.

Analizime devam ediyorum ve IRC loglarına baktığımda herhangi bir IRC bağlantısı görmüyorum. Çoğu botnet yönetimi kolaylaştırdığından oldukça fazla kullanılan bir protokoldür. Diğer protokolleri de incelemeye devam ettiğimde SMTP protokolünün oldukça fazla kullanıldığını görüyorum. W32.Sality, Spam email propagandası yaparak kendisi dağıtma özelliğine sahip olduğunu hatırlıyorum. 10.0.2.113(Host) Ip adresinden yüzlerce IP adresine email gönderilmiş.

![botnet](/../post_images/botnet/bro8.png)

Daha detaylı incelemek istediğimde, fikrimi doğruluyorum.

![botnet](/../post_images/botnet/bro9.png)

BRO'nun en kullanışlı yönlerinden birisi **"weird.log"** dosyası, adından da anlaşılabileceği gibi BRO'nun anormal olarak algıladığı durumları gösteriyor. Buradan ilk olarak dikkatimi çeken şey oldukça fazla **bad_HTTP_request** olması. Yani HTTP request(istek)lerimizin çoğunluğu sorunlu. Oraya tekrar geleceğimizi söylemiştim. Şimdi tekrardan dönelim ve hangi sorunla karşı karşıyayız öğrenmeye çalışalım.

![botnet](/../post_images/botnet/bro10.png)

Şimdi, ilk olarak kötü http isteklerinin neler olduğu görmek istiyorum. Kontrol ettiğimde en fazla "401", "200" ve "404" hatalarının döndüğünü görüyorum.

![botnet](/../post_images/botnet/bro11.png)

Ufak bir not(HTTP durum kodları)

```
200	OK	Tamam
403	Forbidden
404	Not Found	Sayfa Bulunamadı
```
Eğer şuan bir botnet trafiği inceliyor olmasaydık, yine aynı şeyleri söyleyebilirdik değil mi? Sanki birisi bir yerlere oldukça fazla ulaşmaya çalışıyor, deniyor ve bazıları çoktan yok olmuş.

Peki, biraz daha içeri girelim... Nedir bu istekler? User-Agent olarak ne kullanılmış? Hangi URL'lere istek yapılmış?

![botnet](/../post_images/botnet/bro12.png)

Burada oldukça fazla aynı tip bir dizayn olduğunu fark ediyoruz. **"/image//image/logo.gif?[String]"** şeklinde bir düzen var.
Hemen aklıma bazı sorular geliyor. W32.Sality'nin P2P ağı olabilir mi? Burada logo.gif gerçekten bir imaj dosyası mı ?


Trafiğin içerisinden bu dosyaları extract(dışarı çıkartıp) ederek inceleyebiliriz. Fakat burada [**"CapTipper"**](https://github.com/omriher/CapTipper) adında bir yazılımı kullanarak biraz daha güzel bir görüntü görmek istiyorum.

```
python CapTipper.py /home/samet/sality/2014-04-07_capture-win13.pcap
```

![botnet](/../post_images/botnet/cap1.png)

Daha sonra Captipper analizini bitirdiğinde sizi aşağıdaki gibi bir ekran karşılayacak. HTTP trafiği daha iyi analiz edebileceğiz, herhangi bir dosya için virustotal üzerinden kontrol yapabiliriz. Burada bazı dosyaların "BINARY" olarak tespit edildiğini görüyorum. Halbuki biz **"logo.gif"** ile bir imaj dosyası beklemiyor muyduk?

![botnet](/../post_images/botnet/cap2.png)

Bu dosyaları herhangi bir klasöre çıkartmak istiyorum, daha sonra virustotal'e sorarak herhangi bir sorun olmadığını kontrol edeceğim.

```
CT> dump all /home/user/sality/
```
![botnet](/../post_images/botnet/cap4.png)

Tam olarak beklediğimiz gibi, logo.gif aslında bir imaj dosyası değil. Virustotal sonuçları:

![botnet](/../post_images/botnet/cap5.png)

Diğer adreslerden indirilien logo.gif dosyalarını da aynı şekilde virustotal ile sorguladığımda benzeri sonuçlar gördüm. **W32/Sality.B.gen!Eldorado** Sality'nin bir diğer adı. Böylelikle bu dosyaların olağan trafiğimizin içerisindeki sorunlu noktalar olduğu tespit ettik. Peki, şimdi tekrar HTTP trafiğine dönmek istiyorum çünkü trafiğin tamamı bunlardan ibaret değildi. ELSA ortamına dönüyorum.

![botnet](/../post_images/botnet/elsa1.png)

Burada bazı IP adresleri var, bu neyin trafiği diye merak ettiğimde aslında CISCO router giriş sayfaları olduğunu gördüm. Sality, bu sayfalara bazı parolalar deneyerek brute-force saldırısı yapıyordu.

![botnet](/../post_images/botnet/elsa2.png)

Adreslerin çoğuna erişim izni olmadığından herhangi başarılı bir sonuç elde edilememiş, fakat kırmaya çalıştığı adresleri ve parolaları tespit ettim.

![botnet](/../post_images/botnet/elsa3.png)

Klasik default parolaları deneyerek router sayfalarına giriş yapmayı umuyordu. Yakın zamanda çıkan IOT botnetlerin de benzeri yöntemleri kullandığına şahit olmuştuk...

```
support
admin
password
```

Son olarak eklemek istediğim birkaç nokta var, yazıma başlarken oldukça yakın adresler göreceğiz demiştim... Botnet haline gelmiş bazı web sunucuların Türkiye'ye ait olduğunu farkettim ve listeledim. O domainler:

```
aksan.av.tr
kozakoltuk.com
kulaksizhukukburosu.com
erenkarahan.com
www.kapudane.com
datalinksol.com
edirneli.net
emrahkucukkapdan.com

```

## Son Notlar

Suricata için herhangi bir ekleme yapmaya ihtiyaç duymadım, çünkü W32.Sality için EmergingThreats kuralları çoktan yazılmış durumda. Bro ve Suricata/Snort üzerinde pcap dosyasını analiz ettiğinizde kuralların tetiklenmiş olduğunu göreceksiniz. 

Yazı hakkında görüşlerinizi email aracılığı ile ulaştırabilirsiniz.

Kullandığım kaynakların hepsine referans gösterdim, tüm materyallerin sağlayıcılarına teşekkür ederim.

## Kaynaklar:

- [0] https://www.symantec.com/content/en/us/enterprise/media/security_response/whitepapers/sality_peer_to_peer_viral_network.pdf
- https://mcfp.felk.cvut.cz/publicDatasets/CTU-Malware-Capture-Botnet-66-1/
- http://news.softpedia.com/news/Damballa-Enhances-Failsafe-as-P2P-Increasingly-Used-by-Malware-for-C-C-Communications-358583.shtml#sgal_0
- [1] : https://www.welivesecurity.com/2016/08/24/first-twitter-controlled-android-botnet-discovered/
- [2] : https://www.symantec.com/connect/blogs/sality-botnet
- [3] : https://nakedsecurity.sophos.com/2010/07/02/pdf-spam-phones-home-sality/
- [4] : http://blog.checkpoint.com/2015/09/10/analysis-of-the-sality-gambling-campaign/
- [5] : https://kx.cloudingenium.com/microsoft/servers/windows-servers/what-is-www-msftncsi-com/
- [6]http://blog.checkpoint.com/2015/09/10/analysis-of-the-sality-gambling-campaign/
- [7] : https://stratosphereips.org/
- [8] : https://suricata-ids.org/
- [9] : https://www.bro.org/
- [10] : https://securityonion.net/
- [11] : https://tr.wikipedia.org/wiki/Hidra_(efsanevi_yarat%C4%B1k)
