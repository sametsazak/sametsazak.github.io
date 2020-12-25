---
layout: post
title: GoldenEye(Petya) Ransomware Saldırı Analizi
categories: zararli-yazilim some
---

*Bu yazı **[Yavuz Han(@yavuzwb)](https://twitter.com/yavuzwb)** tarafından gönderilmiş olup, **Secops.Blog** tarafından yayınlanmıştır.  

![scilla](/../post_images/petya/scilla.jpg)

*Scylla, Yunan mitolojisinde, Sicilya ve İtalya arasındaki Messina Boğazı'nın durgun tarafından yaşayan canavar.*

# Genel Bakış

Bu yazımda **Petya Ransomware** zararlısına bakacağız. Zararlının ilk çıktığı tahminen 2016 yılının ilk çeyreği içerisinde. Aynı zararlının ikinci versiyonu olan "Green Petya" ise bir yada iki ay sonra Mischa adlı başka bir zararlı ile birlikte kullanılmış olarak ortaya çıktı. Yine birkaç ay sonra zararlının üçüncü versiyonu biraz daha geliştirilmiş olarak piyasaya çıktı. [1]


* TOC
{:toc}

## İnceleme

**Hangi ülkedeki kullanıcılar hedef alındı?**

GoldenEye’nın mailler yoluyla özellikle Almanca konuşan kullanıcıları ve Almanya merkezli şirketlerin HR departmanlarını hedef aldığı belirtiliyor.[2]

## Davranışsal Analiz

Öncelikli olarak zararlıyı Windows 7 Service Pack 1 yüklü sanal ortamda test ediyorum. Açıkcası Windows 8 ve 10’da test etme imkanım olmadı. Zararlıyı indirdikten sonra neler yapabildiğini görmek adına çalıştırmaya karar verdim ve izlemek içinde **Process Explorer** aracını kullandım.

![e](/../post_images/petya/1.png)

Zararlıyı toplamda üç defa çalıştırdım ve her defasında "AppData/Roaming" dizini içerisinde bir klasör(resimdeki ilk adres) oluşturduğunu gördüm ve her defasında bu klasörün içerisine random(rasgele) isimlerde .EXE(çalıştırılabilir) bir dosya oluşturdu.

## Saldırının ilk aşaması

Yüklenen zararlı .exe(çalıştırılabilir) dosyaları otomatik olarak çalıştırılıyor ve kötü amaçlı yazılım harekete geçiyor. 

![e](/../post_images/petya/2.png)

Harekete geçtikten sonra saldırının ilk aşamasında yukarıdaki .txt uzantılı dosya hedefin bilgisayarında açılıyor. Açılan dosyada kullanıcının bilgisayarındaki verileri kurtarması için hangi önergeleri takip etmesi gerektiği yazıyor.

![e](/../post_images/petya/3.png)

Daha sonra ise dosyalara rastgele uzantılar verilerek şifreleniyor. Yukarıdaki resimde görüleceği üzere rastgele uzantı ki bu sanal sürücü üzerinde üçüncü denememdi. Diğer iki denememde daha farklı uzantılar mevcuttu.

## İkinci aşama

![e](/../post_images/petya/4.png) 

Saldırının ikinci aşamasında diğer sürümlerinde olduğu gibi benzer davranışlar sergiliyor ve kötü amaçlı yazılım harekete geçtikten bir süre sonra sistem çöküyor. Bu gerçekleştiği zaman makineye işletim sistemini açtırmıyor ve MBR sürecini değiştirerek makineyi blok ediyor. Bu süreç içerisinde Windows dosya sistemi NTFS’nin bileşeni olan Master File Table(MTB) bileşeni de ekranda CHKDSK adı altına şifreliyor.

Burada zararlı diskteki hataları kontrol ediyor gibi davranıyor fakat işin arka tarafında asıl şifrelemeyi gerçekleştiriyor. Bu işlemin sonunda her zamanki gibi yanıp sönen bir kafatası ile karşı karşıyayız ve sizden bir tuşa basmanızı istiyor. 

![e](/../post_images/petya/5.png)

Zararlı ilk çalıştırıldığında karşımıza çıkan .txt dosyasının tekrardan vücut bulduğunu görüyoruz.

## Kurbanların Yönlendirildiği Sayfa

Zararlının önceki versiyonlarında yönlendirdiği gibi bu seferde yönlendirilen web sayfası benzer bir biçimde karşımıza çıkıyor. Tahmin edileceği gibi Tor üzerinde host edilmiş bir sayfada kurbanların istenilen miktarı sadece bitcoin ile ödemesi gerektiği yazıyor. [1]

![e](/../post_images/petya/6.png)

Fidye ödendikten sonra kurbana verilen Decrypter ile ilk aşamada  kurtarılacak dosyalar belirleniyor.

![e](/../post_images/petya/7.png)

İkinci aşamasında ise Decrypter’ın çalışması için uygun bir anahtarın olması gerekiyor.

![e](/../post_images/petya/8.png)

## Unpacking İşlemi

Burada hızlı bir unpacking işlemi gerçekleştireceğiz. İlerleyen zamanlarda daha detaylı bir şekilde unpacking işlemlerini anlatmaya çalışacağım.İlk olarak indirdiğimiz zararlı içeren dosyamızı debugger’a –ben Immunity Debugger kullandım- atıyorum. 

![e](/../post_images/petya/9.png)

Çalıştırmadan önce ise Debugger Options/Events kısmına giderek Break on new Module(DLL) seçeneğini aktifleştiriyoruz. Böylece yeni yüklenen her modül üzerinde duracaktır. Daha sonrasında ise yüklenen modülleri gözlemleyip, payload’ın hazır olmasını bekleyeceğiz.

![e](/../post_images/petya/10.png)

Sonraki adımda run işlemini başlatıyoruz. Üçüncü defa çalıştırdan sonra Memory Map kısmında gerekli payloadın oluştuğunu fark ediyoruz. Tipik bir kesite ‘.xxxx’ sahip olduğu için kolayca tanıyabildik.

Sonrasında ise backup/save data to file’e tıklayıp masaüstüne kayıt ediyoruz.

![e](/../post_images/petya/11.png)

Kayit ettiğimiz dosya sanal formatta olduğu için düzgün bir şekilde ayrıştırılmayacaktır. Bu yüzden pe_unmapper aracını kullanarak kolay bir şekilde onu raw formatına dönüştüreceğiz. Yukarıdaki resimde gördüğünüz üzere core.dll şeklinde çıktı almak istiyoruz.

pe_unmapper aracını kullanmak için bir bat dosyası oluşturup onu çalıştırcağız. Oluşturacağımız dosyanın parametreleri aşağıda yer alıyor:

```
pe_unmapper <input files> <load base> <output files>
```

![e](/../post_images/petya/12.png)

Daha sonrasında oluşan core.dll dosyasını teyit etmemiz gerekiyor Bu yüzden PE bear adlı toolu açıp Goldeneye core.dll’ni control ediyoruz ve geçerli olduğunu görüyoruz.

![e](/../post_images/petya/13.png)


Unpacking işlemimizi başarıyla gerçekleştirdik. Dll uzantılı dosyamızın geri kalan analizine IDA Pro üzerinden devam edilebilir.

## Dosyalar
**Aşağıdaki dosyalar zararlı yazılım içermektedir. Tüm dosyaları sanal makine üzerinde çalıştırmanızı tavsiye ederiz, bu konuyla ilgili tüm sorumluluk kullanıcıya aittir.**

1-Virustotal (Premium): https://virustotal.com/en/file/b5ef16922e2c76b09edd71471dd837e89811c5e658406a8495c1364d0d9dc690/analysis/

2- Hybrid-Analysis(Free): https://www.hybrid-analysis.com/sample/b5ef16922e2c76b09edd71471dd837e89811c5e658406a8495c1364d0d9dc690?environmentId=100

## Kullanılan Araçlar

Immunity Debugger: https://www.immunityinc.com/products/debugger/
Pe_Unmapper: https://github.com/hasherezade/pe_recovery_tools/tree/master/pe_unmapper
Pe-bear: https://github.com/hasherezade/bearparser/wiki
Process Explorer: http://download.cnet.com/Process-Explorer/3000-2094_4-10223605.html

## Son Notlar

Sadece ‘Kurbanların Yönlendirildiği Sayfa’  kısmında kullandığım fotoğraflar alıntıdır. Diğer fotoğfraflar kendi kurduğum lab’dan alınmıştır.

## Referanslar

Referanslar

1- https://blog.malwarebytes.com/threat-analysis/2016/12/goldeneye-ransomware-the-petyamischa-combo-rebranded/

2- http://www.ibtimes.co.uk/petya-ransomware-variant-goldeneye-targets-hr-departments-by-sending-fake-job-applications-1599659

3- https://www.greekmythology.com/pictures/Myths/Scylla/215442/scylla

