---
layout: post
title: Kimlerin arkasına saklanıyoruz? Türkiye WAF(web application firewall) Araştırması
categories: wiki
---

![logo](/../post_images/waf/logo.jpg)

* TOC
{:toc}


Bu araştırmamda yerli ve milli çözümlerin oldukça sık anıldığı bu zamanlarda Türkiye olarak devlet kurumlarımızın hangi 'web application firewall' ürünlerini kullandığını ve bu ürünlerin ne kadar başarılı olduğunu aktarmaya çalışacağım.

*Not: Bu yazı hiçbir kurumu, kişiyi ve kişileri hedef almamaktadır ve sadece bilgi amaçlı yazılmıştır.* 


# İzlenilen Yol 

Öncelikli olarak araştırmanın yapıldığı adresler:

- Gov.tr uzantılı domain ve subdomain'ler.

1. İlk aşamada bu adresleri Google, Bing, Yandex gibi arama motorlarından elde ettim.

2. Daha sonra wafw00f aracı ile hangi waf ürünün arkasında olduğunu tespit ettim.

Kullandığım yöntem tamamen kullanıcı davranışına uygun herhangi bir (abuse) davranışı bulundurmamaktadır.

# Web Application Firewall nedir?


Web Application Firewall(Web Uygulaması Güvenlik Duvarı) bir web uygulaması ile İnternet arasındaki HTTP trafiğini filtreleyerek ve izleyerek web uygulamalarının korunmasına yardımcı olan sistemdir. 

Genellikle web sitelerini "cross-site forgery, cross-site-scripting (XSS), file inclusion, SQL injection, Layer 7 DDOS" gibi saldırılara karşı korur. WAF, OSI modelinde 7.katmanda çalışmaktadır her tür saldırıya karşı savunmak için tasarlanmamıştır.

Örnek bir waf mimarisi:

![waf](/../post_images/waf/waf.png)
(https://www.maxcdn.com/one/visual-glossary/web-application-firewall/)

# Waf ürünleri ve Başarıları

- F5Networks BIG-IP Ürünü

2.200'den fazla saldırı imzası/kuralı yüklenmiş olarak gelmektedir. Bu kurallar, ağ trafiğine izin vermek veya engellemek için kullanılmaktadır. Çok büyük ağlar için bundle olarak satın alınabilmektedir.

Örnek fiyat listesi aşağıdadır:

![f5](/../post_images/waf/f5fiyat.png)

- Imperva SecureSphere

SecureSphere Web Uygulaması Güvenlik Duvarı, tüm kullanıcıların kritik web uygulamalarınıza erişimini analiz etmektedir. SecureSphere Web Uygulaması Güvenlik Duvarı dinamik olarak uygulamalarınızın “normal”ini öğrenip anormali bulmak üzerine kuruludur.

AWS üzerindeki fiyatı aşağıdaki gibidir:

![f5](/../post_images/waf/impervafiyat.png)

- Citrix Web App Firewall diğer waf ürünleri gibi koruma sağlamaktadır. Bir bulut çözümü olarak mevcut veya Citrix ADC platformuna entegre edilmiş, basitleştirilmiş konfigürasyon kontrolleri varıdr.

Örnek fiyatlar aşağıdaki gibidir:

![f5](/../post_images/waf/citrix.png)


Bu ürünler örnek olarak verilmiştir, fiyatlar google üzerinden bulunmuştur. Ağ yapısına göre ve kullanılacak ürünlere göre fiyatlar değişmektedir.


# Araştırma Sonuçları
---

##  Gov.TR Devlet Kurumları

Öncelikle Google üzerinden elde ettiğim 983 gov.tr uzantılı (domain ve subdomain'ler) ile elde ettiğim sonuçlar aşağıdaki gibidir.

- 983 *gov.tr uzantılı web sitesinden **476**'sının hangi WAF ürününü kullanıldığı tahmin edilmiştir. **Yani %48.42'si tahmin edilmiştir.**

- Toplam 983 gov.tr uzantılı web sitesi ile yaptığım araştırmada bu websitelerinin %48.22'sinin (476 web sitesi) hangi waf ürününü kullandığı tespit edilmiştir.


- 309 Web sitesi Citrix Netscaler arkasında= % 64.91
- 119 Web sitesi F5 BIG-IP arkasında = %25
- 32 Web sitesi ModSecurity arkasında = %6.72
- 3 Web sitesi IBM Web Application Firewall arkasında = %0.6
- 9 Web sitesi Imperva SecureSphere arkasında = 	%1.89
- 2 Web sitesi Cloud Flare arkasında = % 0.42
- 1 Web sitesi Juniper WebApp Secure arkasında = % 0.21
- 1 Web sitesi Better WP Security arkasında = %0.21

**Korunmaktadır.**

![waf](/../post_images/waf/govtr.png)


Bu ürünleri geliştiren ülkeler ve bunların oranları:

- Citrix Netscaler = Fort Lauderdale, Florida, United States
- F5Networks = Seattle, Washington, United States
- JuniperNetworks = Sunnyvale, California, United States
- IBM = Armonk, New York, United States
- Imperva SecureSphere = Redwood Shores, Redwood City, California, United States
- ModSecurity = **Açık Kaynak**, Trustwave, Chicago, Illinois, United States
- CloudFlare = San Francisco, California, United States
- Better WP Security = United States

Üstteki açıklamadan da görüldüğü üzere tüm ürünler Amerika Birleşik Devletler'inde bulunan şirketler tarafından geliştirilmektedir. Bunlara ek olarak ModSecurity açık kaynak olarak geliştirilmektedir.

![waf](/../post_images/waf/govtr_ulke.png)


- Geri kalan %51.78(507) gov.tr uzantılı web sitelerinin hangi waf ürününü kullanıp/kullanmadığı tespit edilememiştir.

# Sonuç

Bu sonuçlara dayanarak gündemde oldukça fazla geçen "yerli ve milli" çözümler içerisinde henüz başarılı bir Web Application Firewall ürünü bulunmadığı ve devlet kurumlarının yabancı menşeili ürünleri tercih ettiğini görmekteyiz.



Samet Sazak
smtszk@gmail.com

---







