---
layout: post
title: Alpha Ransomware Kaynak Kod Analizi ve Unpacking - 1
categories: zararli-yazilim
---

![logo](/../post_images/alpha/logo.jpg)

(Markut (ya da Merküt); Altay efsanelerinde, gök yolculuğuna çıkan kamın ruhuna, ilk üç gökkatı boyunca kılavuzluk eden dev dişi gök kuşudur.)

*Bu yazı **[Yavuz Han(@yavuzwb)](https://twitter.com/yavuzwb)** tarafından yazılmış olup, **Secops.Blog** tarafından yayınlanmıştır.*

Selamlar,

Bugünkü yazımda .Net ile yazılmış Alpha Ransomware’i  unpacking yapacağız. 2016 yılının ortalarına doğru çıkan bu zararlının, Cerber adlı zararlıyı yazan kişi yada grup tarafından yazıldığı tahmin ediliyor. Bazı yerlerde Alfa bazı yerlerde ise Alpha olarak öne çıkıyor fakat bu isim olayını deşmeden direkt olarak Alpha dedim :) Bunun haricinde inceleme iki kısımdan oluşacaktır.


* TOC
{:toc}

# HxD Editör ile kısa bir bakış


İlk olarak indirdiğimiz zararlıyı HxD editör ile açıp bilgi sahibi olabiliyoruz. İndirdiğimiz .bin uzantılı örnek dosyayı HxD ile açtığımızda  inceleyip, dikkatli bakarsak eğer zararlı hakkında ön bilgiler elde edebiliriz. Karşımıza çıkan karmaşada tüm herşeyin dissasemble olduğunu görüyoruz. 


![logo](/../post_images/alpha/resim1.png)

Editörde açılan kısımlara bakmak için aşağıya doğru gittiğimizde  sürüm bilgisi (v2.0.50727) ve  #Strings,  #US, #GUID ve #Blob gibi Net Assembly ile ilgili dört tane stream’e rastlıyoruz. 

![logo](/../post_images/alpha/resim2.png)

Buradan yola çıkarak  .Net assembly ile ilgili olarak binary kodlar içerisinde module tagını aratıyoruz ve karşımıza .Net dosyaları hakkında ipuçları veren ilginç bir kısma rastlıyoruz.

![logo](/../post_images/alpha/resim3.png)


İlgili kısma bakmaya devam ettiğimizde buranın ASCII ile oluşturulduğunu fark ediyoruz. Bunun yanında bu kısımda SymmetricAlgorithm.CreateDecryptor.CreateEncryptor adlı bir dizine rastlıyoruz ve şifreleme için Base64 kullanıldığını fark ediyoruz. 

Bu yapının hemen bitişinde ise strings ile ilgili bir kısma rastlıyoruz. Burada hemen dikkatimizi Unicode Strings’e ait  Load ve GetBytes adlı ikili çekiyor. Bu iki yapı bize  reflection kullanıldığı referansını veriyor. Bu yüzden arama kutusuna gelip Reflection aramasını yapıyoruz ve module yapısının içerisinde kullanıldığını görüyoruz. 

Biraz daha alt taraflara gittiğimizde HTTP Analyzer Stand-Alone adlı bir yapıyı fark ediyoruz. Tahminim her kim HTTP analiz yaparsa bu fonksiyon şifrelemeyi çağıracak ve böylece analiz yapacak kişi ilgili veriye ulaşamayacak. Biraz daha aşağıya doğru indiğimizde ise kalan tüm verilerin şifrelenmiş olduğunu görülüyor ve maalesef herhangi bir çıkarım yapılamiyor.





## DnsSpy ile Kaynak kod incelenmesi

Bu kısma başlarken DnSpy ile birlikte De4dot araçlarına ihtiyacınız olacak. Github üzerinden bu iki aracı indirdikten sonra derlemeye çalıştığımda lib hataları aldım. Sağolsun Twitter’dan 0x94, bu iki aracın derlenmiş versiyonları benimle  paylaştı ve beni büyük bir yükten kurtardı. 

Kendisine buradan bir kez daha teşekkür ediyorum. İsteyen kişiler ile bu iki aracın derlenmiş versiyonlarını paylaşabilirim.

Hybrid-Analysis üzerinden indirdiğim dosyayı ilk olarak DnSpy’da açmaya kalktığım zaman giriş noktasının (entry point) ve değişkenlerin(variables) tam olarak belli olmadığını fark ediyorum. Bu yüzden zararlı dosyamızı daha okunur hale getirmek için De4dot aracını kullanıyorum. Kullandıktan sonra ise zararlı dosyanın  daha okunur hale geldiğini göreceksinizdir.


![logo](/../post_images/alpha/resim4.png)

Video: https://www.youtube.com/watch?v=i1XlSSqbnLQ&feature=youtu.be

Gerekli olanları yaptıktan sonra dosyamızı DnSpy ile açıyoruz. Açtığımız zaman karşımıza gelen ekranda giriş noktamızın ns0.Class.Main referansını kullandığını görüyoruz. 


![logo](/../post_images/alpha/resim5.png)


Bu referansa gittiğimde ilk olarak gözüme zararlı yazılım içersinde görsel dosyaları etkinleştirmesi için Application.EnableVisualStyles(), uygulama genelinde varsayılan değeri ayarlaması için application.setcompatibletextrenderingdefault(false) ve daha sonrasında ise run methoduyla birlikte ZhlioEZcBPuApre class’ını çalıştırıyor ve buradaki sınıf yapılarını alıyor. (Biraz araştırma yaptığımda bahsettiğim methodların proje oluşturma aşamasında .Net’de otomatik olarak geldiğini de öğrendim.)

ZhlioEZcBPuApre classına gittiğimizde ilk olarak InitializeComponent yapısının çağrıldığı gözüküyor. Bu kisma  gittiğimizde this. ZhlioEZcBPuApre_Load’in  çağrıldığı görülüyor. Bunun neleri çağırdığına baktığımızda ilgili kod yapısında Application.Exit() olduğu gözüküyor fakat bundan önce üç tane methodun tanımlandığı gözüküyor. 

![logo](/../post_images/alpha/resim6.png)

method_0()’a gittiğimizde bu yapının çok fazla bir şey yapmadığını ve sadece String’e döndüğünü görüyoruz.  İkinci olarak tanımlanan this.Class2_0.method_0() diğerine nazaran biraz daha fazla birşeyler yaptığı  fakat herhangi bir şekilde return olmadığı gözüküyor.

method_1()’e gittiğimizde ise diğerlerine nazaran daha ilgi çekici gözüküyor. Özellikle kod parçaçaıgının son kısmına baktığımızda her zaman return true’ya döndüğü gözüküyor. Kod parçacığının üçüncü satırında Type type ile başlayan kısmın devamında bulunan kısma incelemek için içten dışa inceleme methodunu kullanacağız. Bu yüzden this.class1_0.method_1’e tıklayıp ilgili kod kısmına gidiyoruz.



![logo](/../post_images/alpha/resim7.png)


Bu kısımda if ile başlayan kısmın bazı şeyleri çağırdığını fark ediyoruz. Buna referans olarakta ilgili kısımda yer alan Load diyebiliriz. Şuan için sadece bunu referans alabiliriz fakat geriye kalan kodları da takip edersek eğer neler yaptığını daha net anlayabiliriz diye düşünüyorum. If kısmının hemen bir önceki satırında bir method oluşturmak için reflection kullanıldığını görüyoruz. Yine If ile başlayan satıra geldiğimizde oluşturulan methodInfo’unun isminin Load olduğunu ve getParameters kullanıldığını görüyoruz. Bu parameter’i kullanarak bir sonraki satırda Load methodunu çağırıyor. Bir sonraki satırda ise byte_0’a geldımizde bunun parameter olarak kullanıldığını ve assembly olduğunun referansını veriyor. Kod parçacığının başında tanımlanan byte_0’ın büyük ihtimalle buradan başlayarak hafızanın içine yerleştiği, daha sonra da bir sonraki yine  assembly olan byte_0’ın çağırdığını zannediyorum.


![logo](/../post_images/alpha/resim8.png)

Bir sonraki adımda ise Byte_0’ın decrypt’ı nereden çağrıldığını bulmak için ZhlioEZcBPuApre classına geri dönüp private string method_0’a bakıyoruz. Bu kod parçacığında Class2.Struct0.byte_1=Class3.F yapısı yer alıyor. Class3.F ‘3 gittiğimizde ise Byte_0’ın decrypt’i nereden çağırdığını görüyoruz. İlgili kod parçasında mKYc.resources’ın çağrıldığı gözüküyor. Hemen yan panelde zararlıya ait klasörlerin yer aldığı kısımda Resources adlı klasör var ve o klasöre gidip  mKYc.resources’ı Desktop’a kayıt edebiliriz veya üzerine sağ tıklayıp Show in Hex Editor’de yapabiliriz. Hex Edıtor ile baktığımızda bazı kısımların encrypt bazı kısımlarında decrypt edildiğini görüyoruz. 


![logo](/../post_images/alpha/resim9.png)


Daha sonra ise Start butonuna basıp Desktop üzerinde yer alan Alpha-cleaned.bin dosyamızı seçiyoruz ve çalıştıdıktan sonra giriş noktasına geri döndüğümüzü göreceksiniz. Daha sonrasında continue diyoruz ve öncesinde breakpointin otomatik olarak koyulduğu - this.class1_0.method_1’e tıklayıp gittiğimiz kod kısmı- yere gidiyoruz.


![logo](/../post_images/alpha/resim10.png)

Bir sonraki adimda ise en alt kısımda Locals altında byte_0, type_0 ve type_1 lerin yer aldığı bir kısım göreceksiniz. İlk seçenekte yer alan byte_0’a tıkladıktan
sonra size gelen ekranda  bu öğrenin 10 binden fazla alt öğe içerdiğini,  görüntüleme kısmında da sınırlı sayıda gösterileceğini belirtip kabul edip-etmeyeceğinizi soruyor ve bizde kabul ediyoruz. Byte_0’a gelip sağa tıklıyoruz ve Show in Memory deyip daha sonrasında gelen ekranda binary dosyasını göreceksiniz ve bunu da Desktop’a kayıt ediyoruz ve stop butonuna basıp durduruyoruz.

Not: Zararlıya ait incelemenin ikinci kısmı daha sonra yayınlanacaktır.


## Zararlı Dosyalar

**Aşağıdaki dosyalar zararlı yazılım içermektedir. Tüm dosyaları sanal makine üzerinde çalıştırmanızı tavsiye ederiz, bu konuyla ilgili tüm sorumluluk kullanıcıya aittir.**


1- Hybrid-Analysis(Free):

https://www.hybrid-analysis.com/sample/fc21ac155de398cba7c44085751c5719b8f404715a0417ffba441296c998c20b?environmentId=100

