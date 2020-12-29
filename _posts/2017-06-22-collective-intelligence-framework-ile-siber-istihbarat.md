---
layout: post
title: Collective Intelligence Framework ile Siber İstihbarat
categories: wiki
---

![logo](/../post_images/cif/goblin.jpg)

(Goblinler, kötü ruhlu, huysuz, zararlı, tuhaf-çirkin vücutlu, cin ahalisinden bir hayalet türüdür, boyu bir cüceninki ile bir insanınki arasında değişik uzunluklarda olabilir.)

* TOC
{:toc}


## Collective Intelligence Framework nedir?

CIF(collective intelligence framework) bir açık kaynak siber tehdit istihbaratı yönetim sistemidir. CIF bilinen yüzlerce siber istihbarat servisinden veri toplamanızı sağlar. Bu veriler IP adresi, FQDN, URL ve herhangi bir zararlı davranışa ait bilgiler olabilir.


Github: https://github.com/csirtgadgets/massive-octo-spice


Genel bir özetle:

![cifv2](/../post_images/cif/cif_overview.png)


Framework'ün çalışma mantığı şu şekildedir.

 - Herhangi bir kaynaktan veri alma.
 - Bu veriyi kaydetme ve reputation(itibar)'a göre değerlendirme.
 - Sorgular aracılığıyla tekrar veriye erişim ve gözlemleme.

# Genel bakış


## Parsing ve Store işlemi

CIF farklı veri tiplerini takip etme yeteneğine sahip, örnek üzerinden gidersek USOM'a ait https://www.usom.gov.tr/zararli-baglantilar/1.html adresinde bulunan tehdit istihbaratı verileri CIF'e girdi olarak verilebilir. Buradaki adresler parse edilerek, URL, IP adres, Açıklama, tarih, güvenilirlik gibi alanların parsing işlemine tabi tutularak ayrı ayrı Elasticsearch veritabanına JSON formatında kaydedilir. Bu veri daha sonra Kibana yardımıyla görselleştirilebilir ve kullanılabilir. Elasticsearh oldukça kolay kullanılabilen bir big data platformudur.

![logo](/../post_images/cif/usom.png)
(USOM tehdit istihbaratı verileri)

## CIF Processleri ve Açıklamaları

CIF farklı işleri yapmak üzere birkaç process kullanmaktadır.

- cif-smrt - veriyi indirme, parsing, normalize etme ve tehdit istihbaratını kaydetme.
- cif-worker - indirilmiş veriyi analiz etme işlemlerini yapar.
- cif-starman - HTTP API servisini başlatır. Bu sayede HTTP aracılığıyla CIF kullanılabilir.
- cif-router - zmq message broker
- ElasticSearch - Veritabanı.

CIF hakkında daha fazla detay için bir standalone kurulum yaparak daha iyi anlatacağım. CIF hakkında detaylı tüm bilgiyi The CIF Book'tan elde edebilirsiniz. (https://github.com/csirtgadgets/massive-octo-spice/wiki/The-CIF-Book) 

## CIF Server Kurulumu

CIF'in tavsiye ettiği kurulum ortamı Ubuntu Server üzerinde:
	

```
x86-64bit platform
en az 16GB ram
en az 8 çekirdek
en az 250GB OS kurulumundan sonra boş alan.

```
Ben bu kurulumu bir CIF'e ait docker container'ını çalıştırarak yapacağım. Bu şekilde minimal bir kurulum yapmış olacağız ve CIF'e ait anlatmak istediğim bazı noktalara değineceğim.

Docker CIF deposu : https://github.com/csirtgadgets/massive-octo-spice/tree/develop/Docker

İlk adım olarak docker hub üzerinden CIF container'ını pull edelim.


![cif](/../post_images/cif/docker1.png)

```
docker pull aeppert/cifv2

```
Daha sonra indirdiğimiz imajı

```
docker run -d -p 8443:443 -p 5000:5000 -p 9200:9200 aeppert/cifv2

```

Diyerek çalıştırıyoruz. Containerımız çalışmaya başlıyor. **"docker ps"** komutu ile container ID'sini öğreniyoruz.
(ab663fd3770c) =  container ID

Burada 8443:443'e bind edilerek 443 üzerinden HTTP API, 5000 portu üzerinden Wsgi ve 9200 üzerinden Elasticsearch üzerine erişmek için kullanılıyor.

(-d : daemon, -p bind ports)

Daha sonra çalışan container'ımızın içerisine girerek, CIF'i kullanmaya başlayabiliriz. İsterseniz github deposu üzerinde "Easybutton" ile ubuntu server'a bir script aracılığıyla CIF'e kurup dökümanı takip edebilirsiniz.


```
docker exec -it ab663fd3770c /bin/bash
```

Container içerisindeyken kök dizin içerisinde "start.sh" adında bir bash script bulunuyor. Bu bash script'i çalıştırarak CIF'i çalıştırıyoruz.

```
./start.sh
```

![cif](/../post_images/cif/docker2.png)

Container'ı çalıştırdığım andan itibaren Elasticsearch, cif-smrt, cif-router, cif-worker, cif-starman ve monit processlerini çalıştırmış.

CIF kullanmaya başlamadan önce göstermek istediğim bazı konfigurasyon ve dosyalar var. "/etc/cif/" dizini altında cif-smrt.yml, cif-starman.conf, cif-worker.yml ve rules dizini bulunmaktadır.

Cif-smrt.yml içerisinden Client IP adresi ve token ayarlarını.

Cif-starman.conf içerisinden TLP map ayalarını.

Cif-worker.yml içerisinden yine token ayarlarına erişebilirsiniz.

Rules dizini içerisinde CIF ile birlikte gelen tehdit istihbaratı kaynaklarının bulunduğu dizindir. 

Default liste aşağıdaki gibidir.

![cif](/../post_images/cif/docker3.png)

Buradan daha detaylı bir açıklama yapmak için emergingthreats.yml dosyasını incelemek istiyorum.

```
vim /etc/cif/rules/default/emergingthreats.yml
```

![cif](/../post_images/cif/docker4.png)

CIF feed olarak bu dizinde bulunan yaml dosyalarının içerisindeki veriye göre hareket ediyor. Örneğin burada provider emergingthreats http://rules.emergingthreats.net/blockrules/compromised-ips.txt dizininde tehdit olarak sunduğu IP adreslerini barındırıyor.

- Provider = Sunucu
- Confidence = Güven oranı
- tlp = green (Anlamlarını cif-starman.conf dosyasında bulabilirsiniz.)
- tags = Belirlenmiş olan tag bu tagleri kullanarak daha sonra sorgu yapacağız.
- pattern = Bu regex pattern'i ile feed olarak verilmiş adresten IP adreslerini parse ediyor.

Bu şekilde daha sonra USOM feedleri için bir konfigurasyon yazacağız. 

## CIF Kurulum Sonrası

CIF kurulumumuzu Docker container'ı ve servisleri başlatarak tamamlamıştık. Bir ön bilgilendirmeden sonra CIF'e ilk sorguları atarak kontrollerimizi yapabiliriz.

Öncelikle logları kontrol ederek başlıyorum.

```
tail -f /var/log/cif-smrt.log
```
Dosyası içerine baktığımda /etc/cif/rules/default dizini içerisinde feed kaynaklarının cif-smrt process'i tarafından işlemeye alındığını görüyorum. Buradaki adreleri tek tek kontrol edip, ilgili regex pattern'i ile parse ediyor ve elasticsearch veritabanına kaydediyor.


![cif](/../post_images/cif/docker5.png)


Daha sonra elasticsearch veritabanını kontrol ediyorum. Bu komut ile elasticsearch içerisine kaydedilen indexleri göreceğim.

```
curl -XGET "localhost:9200/_cat/indices/" 

```

```
yellow open cif.tokens                 5 1     3 0 11.8kb 11.8kb 
yellow open cif.observables-2017.06.22 5 1 18310 0 11.3mb 11.3mb 
```

İlk verilerin kaydedilmeye başladığı görüyorum. CIF default olarak bu processi kısaca fetching(çekme) işlemini 3 saatte bir tekrar başlatıyor, feed kaynaklarından veriyi alıp bir cache üzerinde tutuyor. Daha sonra buradan parsing işlemiyle devam ediyor. Bu döngü sistem kaynaklarına göre artırıp azaltılabilir.

Bir süre sonra kontrol ettiğimde:

```
yellow open cif.tokens                 5 1     3 0 11.8kb 11.8kb 
yellow open cif.observables-2017.06.22 5 1 25396 0 24.2mb 24.2mb 
```

Verilerin gelmeye devam ettiğini görüyorum. Şimdi buradan CIF sorgularını ve neler yapabildiklerine değinelim.

## CIF kullanımı

CIF client'i /usr/local/bin/cif dizininde bulunmaktadır. İlk olarak -h parametresiyle uzun bir çıktıyla yardım alabileceğiniz bir çıktı mevcut.


![cif](/../post_images/cif/docker6.png)

Şimdi Client'e bazı sorgular göndererek verileri elde edeceğiz.

### IP tabanlı sorgular

```
$ cif -q 91.223.133.13
```

Sorgusu ile 91.223.133.13 IP adresine ait herhangi bir kayıt olup olmadığını soruyoruz.

Çıktımız şu şekilde oluyor. Bu IP adresinin bir bot olduğunu "blocklist.de" provider'ından elde edildiğindi. Açıklamasını rapor edildiği tarihi, cc/asn, güvenilirlik seviyesini ve sahip olunan tüm bilgilere erişebiliyoruz. Tek bir IP adresinden elde edilen veri:

![cif](/../post_images/cif/docker7.png)


IP sorguları aşağıdaki gibi de yapılabilir.

```
$ cif -q 130.201.0.0/16
$ cif -q 2001:4860:4860::8888
```


### FQDN sorguları

CIF'e fqdn sorguları da yapılabilir. Herhangi bir domaine ait istihbarat bu şekilde elde edilebilir. Örneğin:


```
cif -q google.com

```
Sorgusunu yaptığımızda çıkan sonuçtan, google.com domaini alexa.com üzerinden top1000 domainler listelenerek whitelist(beyaz liste) içerisinde olduğunu görüyoruz. Google.com güvenilirlik değeri "95" olarak belirtilmiş ve bir tehdit olarak görülmüyor.

![cif](/../post_images/cif/docker8.png)


### URL sorguları

CIF üzerinde herhangi bir URL'in tehdit olup olmadığını da kontrol edebilirsiniz.

```
$ cif -q 'http://www.google.com'
$ cif -q 'https://www.google.com/search?12345.html'
```


### Hash sorguları

CIF üzerinde herhangi bir zararlıya ait, hash kayıtlarını feed olarak verebilir ve aynı zamanda sorgulayabilirsiniz.

```
$ cif -q de305d54-75b4-431b-adb2-eb6b9e546013                              # uuid
$ cif -q 3b6a927c890f067ad524baac9d751480                                  # md5
$ cif -q 57c64d62e79a5b9829e5a902e4a3fb22ff618d89                          # sha1
$ cif -q b712dfc617a327ce948e3341fa4d3f759988c299fcdbc80630f8b3c2c5408be2  # sha256

```

Eğer "3b6a927c890f067ad524baac9d751480" özetine ait bir zararlı yazılım daha önce veritabanına kayıt edilmiş ise size detaylı bir çıktı verecektir.

### Observable Type sorguları

Burası CIF kullanımında en önemli kısımdır. Çünkü "output type" belirterek CIF üzerindeki tüm veriyi alabilir ve diğer sistemler ile entegrasyon sağlayabilirsiniz. Öncelikle --otype parametresi ile alabileceğimiz çıktılara bakalım.


```
$ cif --otype ipv4   # ipv4 adresleri
```

Veritabanında bulunan tüm IPv4 adresleri listeler. Bu sorguya limit uygulamak için --limit parametresi çıktı olarak table, json, csv, snort, bro, bind ve html'i kullanabilirsiniz. Örneğin:

```
cif --otype ipv4 --limit 100 -f csv
```

Çıktı olarak CSV'yi kullandık.

![cif](/../post_images/cif/docker9.png)

```
cif --otype ipv4 --limit 100 -f snort
```

![cif](/../post_images/cif/docker10.png)

Sorgusunu çalıştırdığımızda bize Snort IDS kuralı olarak çıktı verdi. Bu kurallar hiçbir işleme tabi tutulmadan snort kuralı olarak verilebilir. Aynı şekilde BRO IDS için de çıktı alınabilir.

```
cif --otype ipv4 --limit 100 -f bro
```

![cif](/../post_images/cif/docker10.png)

Bu kısım CIF framework'ün en önemli kısmı olduğu düşünüyorum çünkü bu çıktıları nasıl kullanacağınız tamamen size kalmış. Tavsiye edilen CIF framework kullanımı bu çıktıları herhangi bir scripting dili hazırlanmış ufak scriptler aracılığıyla manipule edilmesi. Örneğin buradan elde edilen çıktıları Snort'a kural olarak gönderebilirsiniz.


### Diğer Otype Sorguları

```
$ cif --otype ipv4   # ipv4 address
$ cif --otype ipv6   # ipv6 address
$ cif --otype fqdn   # fully qualified domain address
$ cif --otype url    # url address
$ cif --otype email  # email address

$ cif --otype md5    # md5 hash
$ cif --otype sha1   # sha1 hash
$ cif --otype sha256 # sha256 hash
$ cif --otype sha512 # sha512 hash
$ cif --otype uuid   # uuid hash
```

### Tags Sorguları

Hatırlarlarsanız CIF için ön bilgi verdiğim yerde /etc/cif/default/ dizini altında CIF'in feed kaynaklarına ait yml dosyaları vardı. Buradan verilen kaynağa ait "tag" belirtilmişti ve örneğimizde "malware" tag'i kullanılmıştı. Şimdi --tags parametresi ile tüm sonuçları sorgu olarak verebilir ve sonuç döndürebiliriz. Örneğin:


```
cif --tags malware --limit 5
```


![cif](/../post_images/cif/docker12.png)

5 adet "malware" tag'i ile belirtilmiş tehditleri elde ettik. Çıktı tipini -f parametresiyle istediğiniz gibi belirtebilirsiniz.

Burada en önemli yer bu tag'leri sizin belirleyebilir olmanızdır. Yazının devamında USOM feedleri için bir konfigurasyon dosyası yazacağız ve tekrar sorgularken belirttiğmiz tag'e göre yapacağız.

```
$ cif --tags malware
$ cif --tags botnet
$ cif --tags phishing
$ cif --tags scanner
$ cif --tags zeus
$ cif --tags hijacked
```

### Ülke koduna göre sorgulama

CIF Ülke kodu, ASN gibi verileri kendisi toplar. Bu verilere uygun sorgular yapabilirsiniz.


```
$ cif --cc US
$ cif --cc CN
$ cif --cc JP
```
### ASN sorgulama

```
$ cif --asn 36351
$ cif --asn 199789
```

### Provider'a göre sorgulama

CIF'e kaynak feed verirken yml dosyası içerisinde provider belirtilmişti. Bu provider'a göre sorgu da yapılabilir. Yani örneğin sadece USOM'a ait feedleri sorgularsak:

```
$ cif --provider usom.gov.tr
$ cif --provider dshield.org
$ cif --provider dragonresearchgroup.org
```
Şeklide olabilir. 


### Confidence'a göre sorgulama

-c parametresi ile confidence(güvenilirlik) oranına göre sorgu yapılabilir. Diğer tüm parametrelerle birlikte kullanabilirsiniz. Bu sayede bir düzey belirleme yapılabilir. Örneğin 50-75 arası low, 75-85 arası medium, 85-100 arası high olarak belirlenebilir.

```
$ cif --otype ipv4 -c 95
$ cif --otype fqdn -c 85
$ cif --otype url -c 65
```

### Application'a göre sorgulama

Application sorgulama tıpkı belirtilen tag sorguları gibi kaynak feed konfigurasyon dosyası içerisinde sadece SSH botnet şeklinde belirtirseniz. Bu şekilde de sorgu yapabilirsiniz.


```
$ cif --otype ipv4 --application ssh
$ cif --otype fqdn --application http
```

Bunların dışında zamana göre, gruplara göre, related data'ya göre sorgu yapılabilir. Tüm sorgular hakkında detaylı bilgiler https://github.com/csirtgadgets/massive-octo-spice/wiki/Introducing-the-CIF-client adresinde bulunmaktadır.


## USOM feedlerinin CIF'e kaynak olarak verilmesi

USOM, zararlı bağlantılar listesini RSS olarak veriyor. Şu adresten elde edilebilir. 

https://www.usom.gov.tr/rss/zararli-baglanti.rss

CIF'e bu veriyi verebilmek için bir konfigurasyon dosyası hazırlamamız gerekmektedir. RSS, JSON, XML, TEXT, delimited text ve tüm detaylar şurada bulunabilir.

https://github.com/csirtgadgets/massive-octo-spice/wiki/ParsingFeeds

Ben RSS parsing için hazırlanmış default parsing dosyasını düzenleyerek USOM feedlerini CIF'e aktaracağım.

Hazırladığım konfigurasyon şu şekilde:

usom.yml

![cif](/../post_images/cif/usom2.png)


Daha sonra USOM feedlerini test ettiğimde:

```
sudo su - cif -c "/opt/cif/bin/cif-smrt --testmode -c -d -r /etc/cif/rules/default/usom.yml"
```
![cif](/../post_images/cif/usom1.png)

1570 adet event kaydedildiğini söylüyor. Şimdi konfigurasyonda kullandığımız tag'e göre ya da provider'a göre sorgu yaparak zararlı bağlantılara erişebilirsiniz.

```
cif --provider usom.gov.tr --limit 1000 -f csv
cif --tags zararli-baglanti --limit 100 -f snort
```

Buradan çıktı olarak SNORT'a veya BRO'ya kural aktarımı yapılabilir. CSV şeklinde çıktı alınıp bu veri manipule edilebilir ve diğer sistemlerle entegrasyon sağlanabilir.

# Sonuç

Siber Tehdit istihbaratı son zamanlarda çok önemli olan konulardan bir tanesi. Collective Intelligence Framework bu alanda kullanılan açık kaynak bir çözüm. İncelediğim CIF sürümü v2 olan sürümüdür. Yakın zamanlarda v3 olan Python dili ile yazılan bir versiyonu daha mevcut ve farklı iyileştirmeler var. Bir sonraki yazımda CIF v3'ü de inceleyeceğim. Faydası olması dileğiyle.

Samet Sazak - smtszk@gmail.com

## Referanslar

- https://github.com/csirtgadgets/massive-octo-spice
- http://csirtgadgets.org/
- http://www.elfwood.com/~jrcoffron/Goblin-Warrior.2870379.html

