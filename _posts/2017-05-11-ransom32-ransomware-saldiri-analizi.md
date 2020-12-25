---
layout: post
title: Ransom32 Ransomware Saldırı Analizi
categories: zararli-yazilim some
---

![logo](/../post_images/ransom/pooka.jpg)

(Pooka, Kelt folklöründe yer alan ve adı geçen sayısız periden biridir. [4])

*Bu yazı **[Yavuz Han(@yavuzwb)](https://twitter.com/yavuzwb)** tarafından yazılmış olup, **Secops.Blog** tarafından yayınlanmıştır.*

Günümüz itibariyle pek çok yazılım şirketi iş modeli olarak **SaaS**’ı kullanmaktadır. Haliyle pek çok malware yazarı ve siber dolandırıcı, bu yeni model üzerinde çalışacak zararlı oluşturmaktadır. 2015 yılı boyunca SaaS modelleri üzerinde çalışan Tox, Fakben veya Radamant gibi zararlılar ortaya çıkmıştı. [1] [2]  Bu yazımızda ise 2016 yılının başında çıkan ilk JS ransomware’i olan Ransom32’ye bakacağız.

* TOC
{:toc}

# Genel Bakış

Yazının başında belirttiğim gibi diğer zararlıların aksine Javascript ile yazılmış ilk zaralı yazılımdır. Daha önceki analizimde olduğu gibi bu zararlıyı da Windows 7 Service Pack 1 yüklü sanal makinam üzerinde test edeceğim.


![logo](/../post_images/ransom/resim1.png)

Ransom32 indirildikten sonra otomatik olarak kendiliğinden açılan Winrar arşivi olduğu gözüküyor. Yukarıdaki resimde görüleceği üzere arşivde "Chrome.exe" dosyası yer alıyor. Winrar arşivi açıldıktan sonra dosyalar arasında bulunan Chrome.exe’yi otomatik olarak çalıştıran script harekete geçiyor.

Yukarıdaki resmin sağ tarafında yazan komutlar:

- Setup=Chrome – Dosya arşivden çıkarıldıktan sonra hangi dosyanın çalıştırılacağı belirtiliyor
- TempMode – Dosyaların %TEMP% klasörüne depolanacağını belirtiyor
- Silent=1 – Yükleme işlemi bitene kadar ek olarak herhangi bir pop-up penceresininin açılmayacağı belirtiliyor.

![logo](/../post_images/ransom/resim2.png)

Görüldüğü üzere zararlı çalışmaya başladıktan sonra "AppData\Local\Temp" dizininde kuruldu.

![logo](/../post_images/ransom/resim3.png)


Burada otomatik olarak harekete geçen "Chrome.exe" zararlısı, beraberindeki tüm dosyaları "AppData\Local\Roaming\Chrome Browser" dizini altında toplar. Bu klasörde saklanan dosyalar sadece gerçek zararlıyı çalıştırma adına bir ortam niteliğindedir.

Yukarıdaki dosyalara baktığımızda çeşitli dosyalar içerdiğini görüyoruz fakat  zararlı yazılımın asıl görevini yerini getirmesinden sorumlu dosyalar .JS dosyalarıdır.

## Oluşturulan Script dosyalarına Bakış

"AppData\Local\Temp" dizininde kurulmuş olan klasörde Node.js modülü yer almaktadır. Modülde yer alan dosyalar aşağıdadır.

![logo](/../post_images/ransom/resim4.png)

Package.json; İlk olarak pek çok yapılandırmayı tanımlayan manifesto dosyasına bakıyoruz.

![logo](/../post_images/ransom/resim5.png)


İkinci olarak "index.html" dosyasına baktığımızda, paketin giriş noktasının burada tanımlandığını görüyoruz. (binary.bin dosyası)

![logo](/../post_images/ransom/resim6.png)

"İndex.html" içerisinde tanımlanan "binary.bin" dosyası, buradaki kötü amaçlı yazılımın kalbi konumunda yer alıyor. Burada JavaScript, native bir kod ile derleniyor ve "evalNWBin" fonksiyonunu kullanarak yükleniyor. Diğer tüm bileşenlerde bu binary dosyası içerisinden çağrılıyor.

![logo](/../post_images/ransom/resim7.png)


"Binary.bin" dosyasını incelediğimiz zaman açıkcası zararlının işlevselliği hakkında pek çok bilgi edinebiliyoruz. Bunlardan bazıları;

- Kullanıcıya ait hangi uzantılı dosyların şifrelendiği
- Dosyaları şifrelemek için kullanılan AES yapılandırılması
- Fidye bilgi penceresini biçimlendirmek için kullanılan CSS kodlarını
- Fidye bildirgesinde yer alan notları
- Bitcoin’i nasıl alacağınız ile ilgili talimatlar. Hatta bunun için bir video hazırlayıp Youtube’a koymuşlar.
- Ödeme alındığında görüntülenecek mesaj

gibi pek çok ayrıntıya ulaşılabiliyor.


![logo](/../post_images/ransom/resim8.png)

Kurbanın bilgisayarındaki dosyalar şifrelendikten sonra  bilgilerin ve talimatların yer aldığı pencere gözükür. 

![logo](/../post_images/ransom/resim9.png)

İnternet bağlantısı dahili Tor istemcisiyle birlikte çalışır. Burada  "AppData\Local\Roaming\Chrome Browser" altında yer alan rundll32.exe hareket geçiyor.

![logo](/../post_images/ransom/resim10.png)

Tor istemcisi isim değiştirmiş olarak  rundlll32.exe olarak çalışıyor.

İnternet bağlantım kapalı olduğu için buradan sonra kendi lab ortamımda daha fazla ilerleyemedim. Fakat bir sonraki adımda muhtemelen para ödeme sayfası gelecektir. Ve ödedikten sonra kurbanın dosyaları nasıl kurtaracağı ile ilgili talimatlar olacaktır. İnternet bağlantım kapalı olduğu için buradan sonra kendi lab ortamımda daha fazla ilerleyemedim. Fakat bir sonraki adımda muhtemelen para ödeme sayfası gelecektir ve ödedikten sonra kurbanın dosyaları nasıl kurtaracağı ile ilgili talimatlar olacaktır.


##Son Notlar

Bu zararlı ile ilgili ilk makale Emisoft tarafından yayınlandı. Dosyaların içeriğine baktığımızda her ne kadar Windows tabanlı bir sistem için olsa da zararlıyı oluşturanlar, yapacakları değişiklikler ile Mac ve GNU/Linux kullanıcılarını da rahatlıkla hedef alabileceğini düşünebiliriz. Fakat Bunun yapıldığına dair henüz bir iz yok – varsa da ben duymamış olabilirim.

## Zararlı Dosyalar

**Aşağıdaki dosyalar zararlı yazılım içermektedir. Tüm dosyaları sanal makine üzerinde çalıştırmanızı tavsiye ederiz, bu konuyla ilgili tüm sorumluluk kullanıcıya aittir.**

1- Virustotal(Premium): https://www.virustotal.com/en/file/148458f5b9b2eb4f514faf03abce621f804ab60e2fe642870823a32456ed5482/analysis/1452562509/

2- Hybrid-Analysis(Free):
https://www.hybrid-analysis.com/sample/148458f5b9b2eb4f514faf03abce621f804ab60e2fe642870823a32456ed5482?environmentId=4

## Referanslar

1- http://blog.emsisoft.com/2016/01/01/meet-ransom32-the-first-javascript-ransomware/

2- https://www.bleepingcomputer.com/news/security/ransom32-is-the-first-ransomware-written-in-javascript/

3- https://blog.malwarebytes.com/threat-analysis/2016/01/ransom32-look-at-the-malicious-package/

4- http://www.paranormaltr.com/2013/06/pooka.html
