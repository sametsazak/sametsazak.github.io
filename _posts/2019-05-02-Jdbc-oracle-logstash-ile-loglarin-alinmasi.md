---
layout: post
title: Logstash JDBC ile Veritabanlarından Verilerini Almak
categories: linux wiki logstash
---

![logo](/../post_images/jdbc/wendigo.jpg)

Wendigo, Kızılderili kültürüne dayanan şeytani bir ruhtur. Bazı korkutucu hikayelerde, bu doğaüstü yaratık insanı lanetleyebilir ve onlara sahip olabilir.

(https://www.behance.net/gallery/46554825/wendigo)


* TOC
{:toc}


Bu yazımda logstash yazılımı ile Oracle ve diğer veritabanından nasıl veri alacağımızı(fetch) ve bu verileri elasticsearch'e aktardıktan sonra nasıl görselleştirebileceğimizi aktaracağım. Öncelikle bazı kavramlar hakkında bilgi alalım.

# JDBC Nedir?
---
Java Veritabanı Bağlantısı (JDBC), Java'da yazılmış yazılımları veritabanlarına bağlamak için kullanılan bir API'dir. API size Structured Query Language (SQL) sorguları atmanızı sağlar ve sonuç döndürür.

Jdbc Mimarisi:

![jdbc](/../post_images/jdbc/jdbc1.jpg)

(https://www.tutorialspoint.com/jdbc/jdbc-introduction.htm)

- Peki bu jdbc ne işimize yarayacak? 

Logstash yazılımında bulunan JDBC input plug-in'i ile birlikte logstash'in veritabanlarına erişmesini, belirlediğimiz SQL sorgularını çalıştırmasını ve döndürdüğü verileri elasticsearch'e aktaracağız.

# Logstash JDBC Plug-in'i nedir?

Bu eklenti, JDBC kullanarak herhangi bir veri tabanındaki verileri Logstash'e aktarabilmek için üretilmiştir. SQL sorgularını tek seferlik veya belirli bir cron oluşturarak belirli zaman aralıklarında çalıştırabilirsiniz. Elde ettiğiniz sonuçlar field'lara yansıtılır. (Bunu ilerde açıklayacağım)

Not: Bu eklenti JDBC kütüphaneleri ile birlikte yüklenmemiştir. İstenilen jdbc kütüphanesi, jdbc_driver_library parametresi ile birlikte indirip kullanacağız.

## Logstash JDBC Plug-in'i Nasıl Kurulur?

Herhangi bir kurulum yapmanıza gerek yoktur, Logstash ile birlikte gelmektedir.


# Örnek bir JDBC Bağlantısı Nasıl Yapılır?

MySQL veritabamızdan veri almak istediğimizi varsayalım. Kullanacağımız yapılandırma nasıl olmalıdır?


![jdbc](/../post_images/jdbc/mysql1.png)

İlk olarak, uygun JDBC sürücü kütüphanesini indirip sunucu üzerinde bir yere eklememiz gerekiyor. Ben /opt/ dizinini tercih ediyorum. (bu dosya sisteminizde herhangi bir yere yerleştirilebilir). 

Bu örnekte, user: mysql kullanıcısını kullanarak mydb veritabanına bağlanır ve belirli bir sanatçıyla eşleşen şarkılar tablosundaki tüm satırları sorgular.  Aşağıdaki örnekler bunun için olası bir Logstash yapılandırmasını göstermektedir. 

Bu örnekteki cron örneği jdbc eklentisine her bir dakikada bu sorguyu çalıştırmasını söylemektedir.


Yukarıda ekte verdiğim örnek Logstash input'udur. Veriyi herhangi bir filtrelemeye dahil etmek istemiyorsanız aşağıdaki output ile elasticsearch'e aktarabilirsiniz.

![jdbc](/../post_images/jdbc/mysql2.png)


Veya eğer veritabanından aldığınız sonuçları log dosyasına basmasını istiyorsanız output olarak ;

```
output{
	stdout{
		codec => rubydebug
	}
}
```

Kullanabilirsiniz.

## JDBC Driver'ını nereden indireceğim?

JDBC driver'larını oracle üzerinden indirebilirsiniz:

https://www.oracle.com/technetwork/database/application-development/jdbc/downloads/index.html

## Önemli Parametreler neler?

Bu konfigurasyonda önemli olan bir kısım var, parametre olarak belirlediğiniz favorite_artist kısmını örneğin bir sorguda değişken olarak kullanmak istediğiniz durumlar varsa bunlar için kullanmalısınız. İlerde başka örneklerde oluşturacağız. JDBC_driver_library ve jdbc_driver_class dizinlerini kullandığınız veritabanına göre değiştirmelisiniz. 

Bir diğer konu ise yetki konusu, kullandığınız user/pass bu tablodaki verileri okumak için yetkili olmalıdır.



## Ben birden fazla sorgu kullanmak istiyorum ne yapacağım?

Her bir SQL sorgusu için ayrı bir Logstash yapılandırma dosyaları tanımlamak veya tek bir yapılandırma dosyasında birden fazla ifade tanımlamak mümkündür.

Tek bir Logstash yapılandırma dosyasında birden fazla ifade kullanırken, her ifade ayrı bir jdbc girişi (jdbc sürücüsü, bağlantı ve diğer gerekli parametreler dahil) olarak tanımlanmalıdır.


# Oracle Veritabanından nasıl veri alırım?

Elasticsearch ve logstash'i kurduğumuzu varsayıyoruz. Daha sonra Oracle veritabanı üzerinde bu tabloları okuyabileceğiniz bir user oluşturmanız gerekiyor.

Aşağıda bir örnek konfigurasyon verilmiştir.

![logo](/../post_images/jdbc/code1.png)


Yukarıdaki konfigurasyonda ojdbc dosyanızın dizinini ve HOST_IP:PORT ve service_name'i düzenlemeyi unutmayınız.

- Oracle üzerinde servis ismini nereden buluruz?

https://stackoverflow.com/questions/22399766/how-to-find-oracle-service-name

Bu konuda daha detaylı bilgiye ulaşmak isterseniz, aşağıdaki kaynakları kullanabilirsiniz:

- https://www.oracle.com/technetwork/database/application-development/jdbc/downloads/index.html
- https://qbox.io/blog/migrating-mysql-data-into-elasticsearch-using-logstash
- https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html#plugins-inputs-jdbc-options


Samet Sazak
smtszk@gmail.com

---







