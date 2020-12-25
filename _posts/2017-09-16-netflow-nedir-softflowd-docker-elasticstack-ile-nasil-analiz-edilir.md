---
layout: post
title: Netflow Nedir? Docker, SoftflowD ve ElasticStack ile Nasıl Analiz Edilir?
categories: wiki
---

![logo](/../post_images/netflow/logo.jpg)

(Arçura, Türk ve Çuvaş mitolojisinde Orman Cini. Ormana giren insanları gıdıklayarak gülmekten öldürdüğü söylenir. Kahkahalar atarak ve tokat şaklaması gibi konuşarak insanları çağırır. Bu sese dönüp bakan olursa o kişiye zarar verir.)

* TOC
{:toc}


## Genel Bakış

Bu dökümanda netflow'un ne olduğunu, mimarisini, nasıl çalıştığını, NSM(network security monitoring) uygulanırken nasıl işe yaracağını ve network trafiğini inceleyerek anomali tespiti yapılmasını bir lab ortamı hazırlayarak anlatmaya çalışacağım. Yazıya başlamadan önce birkaç bilgi vermek istiyorum. Bu yazıda netflow toplamak için "Softflowd" yazılımını kullanacağım. Softflowd ile topladığımız netflow kayıtlarını ElasticStack(Elasticsearch - Logstash - Kibana) kullanarak önce "Logstash" yazılımı ile "ElasticSearch" veritabanına göndereceğiz ve daha sonra da "Kibana" ile Elasticsearch veritabanında bulunan netflow kayıtlarını kullanarak görselleştireceğiz. Bir netflow dashboard'u hazırlayacağız. Lab ortamımızda ElasticStack yapısını tek tek kurmak yerine "Docker" ve "Docker Compose" teknolojilerini kullanarak container'lar üzerinde çalıştıracağız ve bazı konfigurasyonlar yapacağız.

## İhtiyaç Listesi

##### [- Docker Kurulumu](https://docs.docker.com/engine/installation/)

##### [- SoftflowD İndirme Adresi](https://code.google.com/archive/p/softflowd/)

##### [- Docker Compose Kurulumu](https://docs.docker.com/compose/)

##### [- Dockerized ELK Deposu](https://github.com/deviantony/docker-elk)

##### [-ByACC İndirme Adresi](http://invisible-island.net/byacc/byacc.html)

## Netflow nedir? Nereden Toplanır? Ne işimize yarar?

![netflow1](/../post_images/netflow/netflow1.png)


Netflow, CISCO tarafından geliştirilen, bir interface(arayüz) üzerinden geçen IP trafik bilgisi toplanılan ağ protokolodür. CISCO tarafından geliştirilse de Linux/BSD/Solaris gibi işletim sistemleri tarafından da desteklenmiştir. Netflow, trafik paketleri içerisindeki belirli alanları bir araya getirerek bir şablon üzerinden sunar. Netflow'un bu yapısı, ağ kullanıcıları, ağ uygulamaları ve yönlendirmeler hakkında bilgi sağlar. Bir sistem yöneticisi Netflow verisini inceleyerek trafiğin kaynağını(source), varış yeri(destination) hakkında bilgi edinebilir. Netflow destekleyen tüm router ve switch cihazları üzerindeki her interface için Netflow kayıtlarını trafiğin analiz edildiği bir "Netflow toplayıcısına" gönderebilir. Netflow ilk çıktığı zamandan beri birkaç farklı versiyonu üretilmiştir. Netflow’un temel çıktısı akış kaydıdır(flow). Akış kaydı biçimlerinden en yenisi Netflow v9 olarak adlandırılmaktadır. Daha önce geliştirilmiş biçimlerden ayırıcı farkı şablon temelli (template-based) olmasıdır. Şablonlar, akış biçimi için geliştirilebilir tasarımlar sağlar, eşzamanlı temel akış kaydı değişimi gerektirmeden Netflow hizmetlerinde uygulanacak gelişmelere izin verir.


### Peki bir flow neye benzer?

CISCO, Netflow versiyon 5 itibariyle 7 alan belirlemiştir. Bu alanlar:

- Kaynak IP adresi
- Hedef IP adresi
- UDP veya TCP için kaynak port (diğer protokoller için 0)
- UDP veya TCP için hedef port, ICMP için tip ve kod (diğer protokoller için 0)
- IP protokolü
- Giriş arayüzü
- IP Hizmet Türü (Type of Service)

(Nfdump yazılımı ile alınan bir çıktı örneği)

```
 Date flow start          Duration Proto   Src IP Addr:Port      Dst IP Addr:Port     Packets    Bytes Flows
 2010-09-01 00:00:00.459     0.000 UDP     127.0.0.1:24920   ->  192.168.0.1:22126        1       46     1
 2010-09-01 00:00:00.363     0.000 UDP     192.168.0.1:22126 ->  127.0.0.1:24920          1       80     1
```


### Netflow Sürümleri

| Netflow Sürümü | İçeriği                                                                                                                            |
|----------------|------------------------------------------------------------------------------------------------------------------------------------|
| Versiyon 1     | İlk uygulama olup IP mask ve AS numaralarını desteklemez.                                                                          |
| Versiyon 2     | Cisco test sürümü olup yayınlanmamıştır.                                                                                           |
| Versiyon 3     | Cisco test sürümü olup yayınlanmamıştır.                                                                                           |
| Versiyon 4     | Cisco test sürümü olup yayınlanmamıştır.                                                                                           |
| Versiyon 5     | En yaygın sürüm olmakla birlikte sadece IPv4 desteği sunmaktadır.                                                                  |
| Versiyon 6     | Cisco tarafından desteği kaldırılmıştır.                                                                                           |
| Versiyon 7     | v5 içeriğine sahip olup Cisco "Catalyst Swtich"lerde kullanılmaktadır.                                                             |
| Versiyon 8     | v5 bilgilerini çeşitli gruplar halinde sunulması sağlanmıştır.                                                                     |
| Versiyon 9     | Şablon temellidir ve güncel yönlendiriciler (router) tarafından desteklenir . MPLS ve IPv6 bilgisi ie BGP next-hop bilgisi içerir. |
| IPFIX          | IETF standardı olup v9 baz alınarak geliştirilmiştir.                                                                              |

Kaynak : [3]

### Netflow Paket Transfer Protokolü

Netflow kayıtları genellikle UDP protokolünü kullanılır. Eğer Router ya da switch kullanılıyorsa tanımlı olan port 2055'dir. Diğer bir yöntem de verinin SCTP (Stream Control Transmission Protocol) protokolü aracılığıyla iletilmesidir. Port ve varış noktası değiştirilebilir. İletimin TCP ile yapılamamasının sebebi TCP protokolünün karakteristiğinde yer alan paketlerin sırasız dağıtımına izin vermemesi özelliğinden kaynaklanmaktadır. Paketlerin sıralı iletimi de gecikme ve ara bellek problemlerine neden olmaktadır.[1] SoftFlowd kullanırken istediğimiz port üzerinden kayıtları gönderebileceğiz.



### Netflow Mimarisi

![netflow1](/../post_images/netflow/netflow2.png)

Tipik bir netflow kayıt üç ana bileşenden oluşur. Bunlar:

Flow Exporter: Paketleri flowlara dönüştürür ve bir veya birden fazla Flow Collector'a gönderir. Bu yazıda exporter ve collector olarak softflowd'yi kullanacağız. 

Flow Collector: Flow exporter'dan gelen kayıtları işlemekle sorumlu. Diske yazmak ve flow exporter'dan gelen kayıtları ön işleme tabi tutabilir.

Analiz Uygulaması: Toplanan flow kayıtları üzerinden botnet tespiti, dns tünel tespiti, ssh saldırıları, trafik profilleme gibi sorulara yanıt alabileğiniz uygulamalar. Tam olarak bu iş için üretilmese de biz bu yazıda Kibana'yı bu amaçla kullanacağız. Netflow kayıtlarını tutmak için Linux/*BSD üzerinde kullanabileceğiniz açık kaynak kodlu bazı yazılımlar mevcut. Nfdump/Nfsen, Softflowd, Ntopng bunlardan birkaçı...

### Netflow Kullanım Senaryoları

Netflow ortalamımızı hazırladık ve kayıtlarımızı tutmaya başladık. Peki ya şimdi ne yapacağız? Neler öğrenebiliriz? Neleri tespit edebiliriz? Öncelikle network istatistiği olarak neler öğrenebileceğimize bakalım.

- Kim ne kadar bandwith kullanıyor? Kullanmaktaydı?
- Normal/Anormal trafiklerimiz nasıl görünüyor?
- A kullanıcısı/A bilgisayarı/A grubu ne kadar lokal olarak/veri gönderdi/aldı?
- Kullandığımız X ürünü network load'ını ne kadar değiştirdi? Beklenmeyen durumlar oluşacak mı? Ürüne boşuna mı para yatırmış olduk?
- Aniden yükselen bu sivri uçlu grafikler bir DOS saldırısına mı maruz kaldığımızı gösteriyor?
- Ağımızda bir botnet mi mevcut? Ne kadar bağlantı kuruluyor, kaç paket gönderiliyor? Hangi port kullanılıyor?
- Ağımızda DNS Tünelleme mi yapılıyor? Kim yapıyor?  
- Birisi SSH brute-force ataklarından başarılı oldu ve bu trafiği basitçe nasıl tespit edebiliriz?
- Bildiğimiz zararlı IP adresleri var, bu IP adresleri ile konuşan bilgisayar/network cihazları kimlerdir?
- En fazla kullanılan portlar nelerdir? En fazla veri akışı hangi IP adresine gidiyor? Kimler bu trafikten sorumlu?
- **Pentagon, Netflow kayıtlarını tutsaydı F-35 uçağına ait 50 Terabayt veriyi sızdırıldığını fark edebilecek miydi?** [4]
 
CISCO, 2001 yılında direkt olarak DSL ve ISDN erişim harcamaları arttığında ve global olarak kendi çalışanlarını VPN kullanımına geçirdi. Fazladan bir kapasiteye ihtiyaç duyup duymadığını Netflow ve açık kaynak araçlar kullanarak trafiği analiz ettiğini söyledi ve 22.000 kullanıcıyı 3 ay içerisinde VPN'e taşıdı. [5] Bunların dışında yetkisiz WAN trafiğini tespit etme, WAN trafiğinin giderek artması sonucunda trafiği düşürme gibi çalışmalar yaptı. VPN trafiğini inceleyerek uzak(remote) çalışanlarını daha iyi anlamak için netflow kayıtları üzerinde çalışmalar yaptığını söyledi. Burada son soru hariç, netflow kayıtlarından birçok soruya cevap bulabiliriz. Netflow'un sağladığı en önemli etkenlerden birisi Network Security Monitoring yaparken harcanılan parayı da düşürmesidir. Router ya da switch'te yapacağınız bir konfigurasyon ile aktifleştirebilir, açık kaynak kodlu yazılımlar kullanarak bu veriyi analiz edebilirsiniz. Bunların dışında Port mirroring yaparak, bir Linux sunucu üzerinde Netflow kaydı tutabilir ve yedekleyebilirsiniz.

## Netflow Lab Ortamı Kurulumu

![netflow1](/../post_images/netflow/netflow3.png)

Netflow hakkında bilgi verdikten sonra artık lab ortamımızın kurulumuna geçebiliriz. Ben host makine olarak Centos7-64bit bir işletim sistemi ve üzerinde Docker ve docker-compose kullanacağım.

Docker kurulumu farklı işletim sistemlerine göre değiştiği için [şuradan](https://docs.docker.com/engine/installation/) adım adım takip edebilir ve kurabilirisiniz. 

Docker-Compose, Multi container uygulamalar için bir araçtır. Docker üzerinde elasticsearch, logstash ve kibana yazılımlarının olduğu bir multi-container çalıştıracağız. 

Docker-compose kurulumunu pip aracılığıyla yapabilirsiniz.

```
$ pip install docker-compose
```
Şimdi Docker ve Docker-compose kurulumlarımızı tamamladığımızı göre şu github deposundan @deviantony adlı kullanıcının hazırladığı docker-elk deposunu çekebiliriz ve düzenleyebiliriz.

```
$ git clone https://github.com/deviantony/docker-elk
Cloning into 'docker-elk'...
remote: Counting objects: 837, done.
remote: Total 837 (delta 0), reused 0 (delta 0), pack-reused 837
Receiving objects: 100% (837/837), 176.06 KiB | 439.00 KiB/s, done.
Resolving deltas: 100% (307/307), done.

$ ls -l

total 36
-rw-r--r-- 1 root root   881 Eyl  8 15:06 docker-compose.yml
drwxr-xr-x 3 root root  4096 Eyl  7 11:43 elasticsearch
drwxr-xr-x 3 root root  4096 Eyl  7 11:43 extensions
drwxr-xr-x 3 root root  4096 Eyl  7 11:43 kibana
-rw-r--r-- 1 root root  1082 Eyl  7 11:43 LICENSE
drwxr-xr-x 4 root root  4096 Eyl  7 11:43 logstash
-rw-r--r-- 1 root root 10263 Eyl  7 11:43 README.md

```
**Logstash Portunun Ayarlanması**

![netflow1](/../post_images/netflow/netflow5.png)

Burada önemli olan 2 dosya var. Birincisi docker-compose.yml bu dosya içerisinde container'lara ait konfigurasyonlar mevcut. Softflowd'nin çıktı olarak Logstash'e göndereceği UDP portunu belirleyeceğiz. Daha sonra Logstash konfigurasyon dosyası içerisinden input portunu düzenleyeceğiz ve elasticsearch'e göndereceğiz.

docker-compose.yml

```
version: '2'

services:

  elasticsearch:
    build: elasticsearch/
    volumes:
      - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"
    networks:
      - elk

  logstash:
    build: logstash/
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    ports:
      - "12345:12345/udp"
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    build: kibana/
    volumes:
      - ./kibana/config/:/usr/share/kibana/config
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

networks:

  elk:
    driver: bridge
```


Burada yaptığımız ilk değişiklik logstash container'ının dinlediği portu değiştirmek oldu. **"12345:12345/udp"** diyerek logstash container'ın host makinemin 12345 portunu dinlemesini sağlamak.

**Logstash Konfigurasyonunun Yapılması**

![netflow1](/../post_images/netflow/netflow4.png)

"logstash/pipeline/logstash.conf" dosyasını açıyoruz, burada input olarak udp 12345 portundan veri alıp, elasticsearch'e yazmasını ayarlayacağız.

logstash.conf

```
input {
  udp {
    port => 12345
    codec => netflow
  }
}
## Add your filters / logstash plugins configuration here

output {
	elasticsearch {
		hosts => "elasticsearch:9200"
	}
}
```

Her şey hazır... Şimdi container'larımızı çalıştırabiliriz ama ondan önce Softflowd kurulumunu yapacağız.

### SoftFlowd Kurulumu

Softflowd'yi kurmadan önce byacc kurulumunu yapmamız gerekiyor. Şu [adresten](http://invisible-island.net/byacc/byacc.html#download) Download kısmından indirebilirsiniz. 

```
$ tar -xvf byacc.tar.gz
$ cd byacc-20170709
$ $ ./configure
$ make
$ sudo make install
```
Adımlarını tamamladıktan sonra, Softflowd son sürümünü şu [adresten](https://code.google.com/archive/p/softflowd/downloads) indirebilir ve kurulumu aşağıdaki gibi yapabilirsiniz. Benim de kullandığım sürüm 0.9.9.

```
$ cd softflowd-0.9.9
$ ./configure
$ make
$ sudo make install

```

**SoftFlowd'nin Test Edilmesi**

Softflowd'yi başlatarak ve daha sonra tcpdump ile trafiği dinleyerek flow kayıtları üretiliyor mu diye kontrol etmek istersek eğer:

```
$sudo ./softflowd -i wlp4s0 -n 127.0.0.1:12345 -v 9 -d

```

Burada -i parametresi dinlenilecek olan interface'i belirtiyor. -n parametresi host:port şeklinde flow kayıtlarının gönderileceği adres -v versiyon bilgisi ve -d daemon olarak çalıştırmaması için gerekli.

Şimdi softflowd'yi çalıştırdıktan sonra tcpdump ile flow kayıtları üretiliyor mu diye bakalım.

```
$ sudo tcpdump -n -i lo port 12345

03:37:01.869160 IP 127.0.0.1.38461 > 127.0.0.1.12345: UDP, length 264
03:38:01.095815 IP 127.0.0.1.38461 > 127.0.0.1.12345: UDP, length 104

```

Paketler gözükmeye başladığında veriler 12345 portuna aktarılıyor demektir.
Şimdi bu adımları tamamladıktan sonra hem ElasticStack containerlarını hazırlamış olduk hem de Softflowd kurulumunu tamamladık. Şimdi motora gaz vermeye başlabiliriz artık:)


### SoftFlowd ve ELK ortamının çalıştırılması

Kurulumlarımızı tamamladık ve testlerimizi gerçekleştirdik. Şimdi docker-compose kullanarak ELK ortamımımı ayağa kaldırıyoruz.

```
$docker-compose up

```

ELK ortamı ayağa kalktıktan sonra Softflowd'yi çalıştırıp netflow kayıtlarını toplamaya başlayabiliriz.

```
$sudo ./softflowd -i wlp4s0 -n 127.0.0.1:12345 -v 9 -d

```
Kayıtlarımızın Elasticsearch'e aktarıldığından emin olmak için:

```
$ curl -XGET "localhost:9200/_cat/indices/"

```
Elasticsearch indexlerine bakıyoruz... logstash-[timestamp] şeklinde bir kayıt oluşmaya başladıysa, logstash softflowd yazılımından netflow kayıtlarını alıp elasticsearch ortamına aktardığını anlamına gelir.


### Kibana Dashboard'ı oluşturmak

Şimdi Malware Capture Facility'den aldığımız normal trafik ve botnet trafiğini bulunduran pcap dosyasını softflowd kullanraak Logstash aracılığıyla Elasticsearch veritabanına gönderiyoruz.

```
sudo ./softflowd -r botnet-capture-20110810-neris.pcap -n 127.0.0.1:12345 -v 9
```

-r parametresi ile pcap dosyasının adresini gönderiyoruz.

```
softflowd v0.9.9 starting data collection
Exporting flows to [127.0.0.1]:italk
Shutting down after pcap EOF
Shutting down on user request
Number of active flows: 0
Packets processed: 322248
Fragments: 0
Ignored packets: 906 (906 non-IP, 0 too short)
Flows expired: 14732 (6402 forced)
Flows exported: 14732 in 1681 packets (0 failures)

Expired flow statistics:  minimum       average       maximum
  Flow bytes:                  46          3294       2471563
  Flow packets:                 1            22         52540
  Duration:                  0.00s       407.04s     16981.29s

Expired flow reasons:
       tcp =         0   tcp.rst =         0   tcp.fin =         0
       udp =         0      icmp =         0   general =         0
   maxlife =         0
over 2 GiB =         0
  maxflows =      6402
   flushed =      8330

Per-protocol statistics:     Octets      Packets   Avg Life    Max Life
           icmp (1):          60845          757    1067.62s   11102.42s
           igmp (2):            184            4       0.84s       0.84s
            tcp (6):       40457986       235622     120.43s   16303.38s
           udp (17):        8011171        85865    1540.54s   16981.29s

```

Paketler elasticsearch veritabanına yazıldı.

![k1](/../post_images/netflow/k1.png)

Şimdi Kibana üzerinden dashboard oluşturmak için önce bazı ""Visualization(görselleştirme)" araçları kullanacağız. Loglarımızı gördükten sonra sol menüden "Visualize" diyerek ""Create New Visualization" diyerek loglarımızı kullanacağımız görseller ekliyoruz.

Bu konu başlı başına bir **blog yazısı** olduğu için sadece Kibana'nın hazırladığı kendi dökümanını paylaşacağım ve ben Netflow kayıtlarından faydalanmak için bir dashboard hazırlayacağım.

Kibana Guide : https://www.elastic.co/guide/en/kibana/current/visualize.html


Hızlıca oluşturduğum Dashboard

![d1](/../post_images/netflow/dash1.png)
![d2](/../post_images/netflow/dash2.png)

# Sonuç

Buraya kadar Softflowd ile ELK ve Docker kullanarak netflow kayıtlarını Elasticsearch veritabanına kaydettik. Elbette ki şuanda oluşturduğum dashboard çok yeterli bir dashboard değil fakat ikinci yazıda daha detaylı güzelce bir dashboard hazırlayıp malware capture facility projesinden alınmış botnet pcap'ini tekrar yürüterek analizini yapacağız. 

İkinci yazıda görüşmek üzere.

Samet Sazak - smtszk@gmail.com

## Referanslar
- [logo] https://i.pinimg.com/736x/05/2a/1b/052a1bf928430cd426895a329c6144bb--male-fairy-fairy-drawings.jpg
- [1]http://www.bilgiguvenligi.gov.tr/ag-guvenligi/ag-akis-kayitlarinin-siber-guvenlikte-kullanimi.html
- [2]http://www.telecom.otago.ac.nz/tele301/student_html/netflow.html
- [3]http://bidb.itu.edu.tr/seyirdefteri/blog/2013/09/07/netflow
- [4]https://www.rt.com/news/223947-snowden-pentagon-china-hack/
- [5]https://www.cisco.com/c/en/us/about/cisco-on-cisco/enterprise-networks/network-data-monitoring-reporting-web.html