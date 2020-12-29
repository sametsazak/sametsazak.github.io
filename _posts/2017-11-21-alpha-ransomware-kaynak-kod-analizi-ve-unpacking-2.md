---
layout: post
title: Alpha Ransomware Kaynak Kod Analizi ve Unpacking - 2
categories: zararli-yazilim
---

![logo](/../post_images/alpha2/gigant.jpg)

(Gigantlar, yunan mitolojisinde devasa büyüklükteki yılan ayaklı devlerdir. Boyları 12 metredir.)

*Bu yazı **[Yavuz Han(@yavuzwb)](https://twitter.com/yavuzwb)** tarafından yazılmış olup, **Secops.Blog** tarafından yayınlanmıştır.*

İlk yazı: [Alpha Ransomware Kaynak Kod Analizi ve Unpacking - 1](http://secops.blog/zararli-yazilim/2017/10/16/alpha-ransomware-kaynak-kod-analizi-ve-unpacking.html) 


Herkese Selamlar,

Alpha ransomware’ le ilgili kod analizimize kaldığımız yerden devam ediyoruz. Hatırlayacagınız gibi en son binary dosyamızı Desktop’a kayıt etmiştik. Hemen sonrasında ise kayıt ettiğimiz dosyayı DnSpy ile açıyoruz.

![res1](/../post_images/alpha2/resim1.png)

 Dosyayı açtığımızda hemen yandaki panelden Dll dosyası olduğunu fark ediyoruz. Göründüğü kadarıyla debug işlemini direkt olarak gerçekleştiremeyecğiz gibi gözüküyor. Çünkü dll dosyasına baktığımızda entry point’in tam olarak nerede olduğunu bilmiyoruz bu yüzden code execution’un nereden devam ettiğini de bilemeyeceğiz. Bu yüzden yan panelde açılan dosyalara bakacağız.

Panele bakmaya başladığımda Resources altında Settings’in olduğunu fark ediyoruz. Buraya baktığımzda ilk satırda “Antis” ile başlayan bu satırın olduğu ve bu satırında anti Debugging, AV, disassembly vs gibi anti ile ilgili ne varsa başlangıçta aktif olması için yazıldığını görüyoruz. Yine panelden lib dosyalarına baktığımızda Settings ile ilgili kodlara rastlıyoruz.

![res1](/../post_images/alpha2/resim2.png)

BullWorker altındaki kodlara baktığımızda Antis ve Settings ile ilgili kodların olduğunu görebiliriz. Buraya baktığımızda Anti Sandbox, Wireshark ve VirtualMachine ilgili önlemler alındığını farkediyoruz. 

![res1](/../post_images/alpha2/resim3.png)

AntiVM ile ilgili kod bölümüne gittiğimde VirtualBox Guest Additions ile ilgili bir kısma rastlıyoruz. Anladığım kadarıyla zararlı  VB Guest Additions’ın yüklü olduğu fark ettiği andan itibaren çalışmasını durduracak. Yine aynı part içerisnde VMWare ile ilgili yüklenmiş bir tools fark ettiğinde çalışmasını durduracak. Biraz daha incelediğimde kodu yazanlar VBBox’a kıyasla VMWare üzerinde inceleyecekler için daha fazla  önlem aldığı fark ediliyor.

Tekrardan BullWorker’a ait class’a gittiğimizde başlangıçtaki GIZxh gizxh = new GIZxh() bulunan kod parçacığındaki new GIZxh()’a gidiyoruz. Burada Windows System’ ler için bazı import’ların olduğunu görüyoruz. Biraz daha aşağıya indiğimzde ise XOREncryptDecrypt ile ilgili ilginç bir kısım bulunuyor. İlgili kod parçacığında Resources kısmında yer alan File’ı çağıran bir kısım yer alıyor. File kısmında yer alan dosyanın Hex Editor ile incelemesi yapılabili fakat bu kısımdaki işimizi şimdilik bu kadar diyebilirim. Şuan için daha önceden kayıt ettiğimiz Alpha-Cleaned.bin dosyamızın incelemesine tekrardan geri dönüyoruz. Hatırlayacağınız gibi daha önceden gerekli BreakPoint’leri koymuştuk. Dosyaya geri dönüp start butonuna basıp, sonrasında continue butonuna basıyoruz.

![res1](/../post_images/alpha2/resim4.png)


İlgili kod parçacığına geldiğimizde hemen alt tarafta method_2 ile ilgili bir kısım yer alıyor. Kod parçacığına baktığımızda GetBytes ve Invoke ile ilgili bir kısmı görüyoruz. Hangi veya nasıl bir byte aldığını bulma adında buraya bir breakpoint koyuyoruz ve ve işlemi tekrardan başlatıyoruz. Hemen alt tarafta locals kısmının altında kod parçacığındaki char_0’ın olduğa bölüme rastlıyoruz. Char_0’a baktığımzda alt alta sıralanmış “wg4JjFL4Q” dizinine rastlıyoruz. Ne olduğunu veya ne anlam ifade ettiğini bilmediğimiz için birşeyler bulabilmek adında arama kısmından aratacağız.

![res1](/../post_images/alpha2/resim5.png)


İlgili aramayı yaptığımızda herhangi birşey bulunamadı. Bu sefer ise bir önceki işlemeri yaptığımız dosyamızda aratıyoruz. Aratırken ise arama seçenkleri arasındaki Number/String’i seçerek aratıyoruz. Arama işlemi sonucunda InitializeComponent diye birşey bulduğunu görüp ilgili class’a gidiyoruz. “wg4JjFL4Q” dizinin Ns1 altındaki class’ta tanımlandığını buluyoruz. Biraz daha kodlara baktığımda pek fazla ilerlenebilecek gibi gözükmüyor açıkcası. Bu yüzden start butonuna basıp en son breakpoint koyduğumuz noktaya geri dönüyoruz.


![res1](/../post_images/alpha2/resim6.png)

Daha sonrasında ise en son koyduğumuz breakpoint’I kaldırıp yolumuza devam ediyoruz. Şuan için kaldıımız yeri tekrardan söylemek gerekirse Class1 dosyasının 15-37 satırları arasındayız ve burada daha önceden yer alan 2 tane breakpoint bulunuyor. Daha sonrasında ise birinci breakpoint noktasından itibaren stepover(F10) yapıyoruz. Bu işleme başlamadan hemen önce injection partı daha iyi görme adına işlemlerimizi görme adına Process Explorer’ı açıp, devam ediyoruz. 

![res1](/../post_images/alpha2/resim7.png)

Daha sonrasında ise bir defa daha stepover yapıyoruz ve geldiğimiz noktaya baktığımızda private bool method_1()’i görüyoruz. Bir adım sonrasında ise method_1’e gidiyoruz. Bu kısımda bazı types’ların return olduğunu görüyoruz. Bir önceki adıma geri dönüp stepover yapıyoruz. Bu kısma baktığımızda ise true’ya döndüğünü görüyoruz ve sonrasında 20-36 satırları arasında method_1 olduğu takdirde uygulamanın exit yaptığını fark ediyoruz. 

Stepover yaptıktan sonraki kısımda this.class1_0.method_0’dan devam ediyoruz. Bu kısmın dll’den bazı fonksiyonları çağırdığını zannediyoruz ve stepInto(F11) yapıp neler yaptığına bakıyoruz. Gerekli adımdan sonra baktığımızda Base.Text’e return yapıyor. Biraz daha fazla detay için tekrardan stepInto yapıyoruz. Fakat elle tutulur bir şey elde edemediğimiz görüyoruz. Bu yüzden stepout yapıyoruz ve breakpoint’lerin olduğu noktaya geliyoruz. 

Şuan için bu ransomware ile ilgili yaptığım kod analizi toplamda bu kadar diyebilirim. Bu yüzden bu serinin sonu bu diyebilrim. Zararlıya ait kod analizini daha da ilerletmek isteyenler ilk yazıda yayınladığım zararlıyı indirebilir ve daha da ileriye götürebilirler. İlk kaynak kod analizim olduğu için yanlışlarım olabilir bu yüzden düzeltilmesi gereken yerleri yorum olarak yazabilirsiniz.

Herhangi bir değişiklik olmazsa bundan sonraki yazımda ‘Inline Hookings’’ ile yaptığım araştırmamı ve tabi ki denediklerimi aktarmak olacaktır. Veya sizlerin bir önerisi olursa o konuyla ilgili de olabilir.


## Zararlı Dosyalar

**Aşağıdaki dosyalar zararlı yazılım içermektedir. Tüm dosyaları sanal makine üzerinde çalıştırmanızı tavsiye ederiz, bu konuyla ilgili tüm sorumluluk kullanıcıya aittir.**


1- Hybrid-Analysis(Free):

https://www.hybrid-analysis.com/sample/fc21ac155de398cba7c44085751c5719b8f404715a0417ffba441296c998c20b?environmentId=100

## Kaynak

Logo : http://www.hellenica.de/Griechenland/Mythos/Bild/GiantsKircher.jpg


