# TCP Wrappers

TCP Wrappers, kullanılacak servislere kısıtlama getirmek için kullanılmaktadır. Hangi servislere hangi clientların (istemcilerin) erişebileceğini ya da engelleneceğini belirlemektedir.

TCP Wrappers paketleri (tcp_wrappers ve tcp_wrappers-libs) default olarak yüklenir ve ağ hizmetlerine host-based erişim kontrolü sağlar. Paketteki en önemli bileşen /lib/libwrap.so veya /lib64/libwrap.so kitaplığıdır. Genel anlamda, TCP-wrapped hizmeti, libwrap.so kitaplığına karşı derlenmiştir.
TCP-wrapped hizmetine bağlantı girişiminde bulunulduğunda, hizmet ilk olarak client'in bağlanmasına izin verilip verilmediğini belirlemek için host’un erişim dosyalarına (/etc/hosts.allow ve /etc/hosts.deny) başvurur. Çoğu durumda, talep eden client'in adını ve istenen hizmeti / var / log / secure veya / var / log / messages dizinine yazmak için syslog daemon'u (syslogd) kullanır.
Bir client'in bağlanmasına izin verilirse, TCP Wrappers istenen hizmete olan bağlantının denetimini serbest bırakır ve client ile sunucu arasındaki iletişimde başka bir rol almaz.
Erişim kontrolü ve logging’e ek olarak, TCP Wrappers, istenen network hizmetine olan bağlantının kontrolünü reddetmeden veya serbest bırakmadan önce client ile etkileşim için komutlar yürütebilir.
TCP Wrappers, herhangi bir sunucu yöneticisinin güvenlik araçları deposuna değerli bir katkı olduğundan, Red Hat Enterprise Linux içindeki çoğu network hizmeti libwrap.so kitaplığına bağlıdır.
Bir network hizmeti ikili dosyasının libwrap.so ile bağlantılı olup olmadığını belirlemek için, kök kullanıcı olarak aşağıdaki komutu yazın:

`ldd <binary-name> | grep libwrap`

<binary-name> 'i network hizmeti ikili dosyasının adıyla değiştirin. Komut çıktı olmadan doğrudan komut istemine dönerse, network hizmeti libwrap.so ile bağlantılı değildir.
Aşağıdaki örnek, / usr / sbin / sshd'nin libwrap.so ile bağlantılı olduğunu gösterir:
  
  `~]# ldd /usr/sbin/sshd | grep libwrap`
  ` libwrap.so.0 => /lib/libwrap.so.0 (0x00655000)`
  
## TCP Wrappers'ın Avantajları
•	Hem client hem de wrapped network hizmetine saydamlık-Hem bağlanan client hem de wrapped network hizmeti, TCP Wrappers'ın kullanımda olduğunun farkında değil. Banlanmış clientler’den gelen bağlantılar başarısız olurken meşru kullanıcılar oturum açar ve istenen hizmete bağlanır.
•	Birden çok protokolün merkezi yönetimi- TCP Wrapper'ler, korudukları network hizmetlerinden ayrı olarak çalışır ve birçok sunucu uygulamasının ortak bir erişim kontrolü yapılandırma (configuration) dosyası kümesini paylaşmasına izin vererek daha basit yönetim sağlar.

## TCP Wrappers Yapılandırma (Configuration) Dosyaları
Bir client'in bir hizmete bağlanmasına izin verilip verilmediğini belirlemek için TCP Wrappers, genellikle hosts erişim dosyaları olarak adlandırılan aşağıdaki iki dosyaya başvurur:
• `	/etc/hosts.allow `
•	` /etc/hosts.deny `
Bir TCP-wrapped hizmeti bir client isteği aldığında, aşağıdaki adımları gerçekleştirir:
1.	/Etc/hosts.allow'a başvurur - TCP-wrapped hizmeti, /etc/hosts.allow dosyasını sırayla ayrıştırır ve bu hizmet için belirtilen ilk kuralı uygular. Eşleşen bir kural bulursa bağlantıya izin verir. Değilse, bir sonraki adıma geçer.
2.	/Etc/hosts.deny'ye başvurur - TCP-wrapped hizmeti, /etc/hosts.deny dosyasını sırayla ayrıştırır. Eşleşen bir kural bulursa, bağlantıyı reddeder. Değilse, hizmete erişim izni verir.
Aşağıdakiler, network hizmetlerini korumak için TCP Wrappers kullanırken dikkate alınması gereken önemli noktalardır:
•	Önce hosts.allow'daki erişim kuralları uygulandığından, hosts.deny'de belirtilen kurallara göre önceliklidirler. Bu nedenle, hosts.allow'da bir hizmete erişime izin veriliyorsa, hosts.deny'de aynı hizmete erişimi engelleyen bir kural yok sayılır.
•	Her dosyadaki kurallar yukarıdan aşağıya okunur ve belirli bir hizmet için ilk eşleştirme kuralı uygulanan tek kuraldır. Kuralların sırası son derece önemlidir.
•	Her iki dosyada da hizmet için kural bulunmazsa veya her iki dosya da yoksa, hizmete erişim izni verilir.
•	TCP-wrapped hizmetleri, kuralları hosts erişim dosyalarından önbelleğe almaz, bu nedenle, hosts.allow veya hosts.deny'de yapılan herhangi bir değişiklik, network hizmetlerini yeniden başlatmadan hemen etkili olur.

### Erişim Kurallarını Biçimlendirme
Hem /etc/hosts.allow hem de /etc/hosts.deny biçimleri aynıdır. Her kural kendi satırında olmalıdır. Kare (#) ile başlayan satırlar veya boş satırlar göz ardı edilir.
Her kural, network hizmetlerine erişimi kontrol etmek için aşağıdaki temel biçimi kullanır:
`<daemon list> : <client list> [: <option> : <option> : …] `
•	<daemon list> - Procces adlarının (hizmet adları değil) veya ALL wildcard’ların virgülle ayrılmış listesi.
•	<client list> - Kuraldan etkilenen host'ları tanımlayan host adları, host IP adresleri, özel pattern'lar veya joker karakterlerin virgülle ayrılmış bir listesi.
•	<option> - Kural tetiklendiğinde gerçekleştirilen isteğe bağlı bir eylem veya iki nokta üst üste işaretiyle ayrılmış eylemler listesi. option field, genişletmeleri destekler, shell komutlarını başlatır, erişime izin verir veya reddeder ve logging davranışını değiştirir.
Aşağıda temel bir örnek hosts erişim kuralı verilmiştir:
`vsftpd : .example.com `
Bu kural, TCP Wrappers'a example.com domain'deki herhangi bir host'tan FTP daemon (vsftpd) bağlantılarını izlemelerini söyler. Bu kural hosts.allow'da görünürse, bağlantı kabul edilir. Bu kural hosts.deny'de görünürse, bağlantı reddedilir.
Sonraki örnek hosts erişim kuralı daha karmaşıktır ve iki option field kullanır:
`sshd : .example.com  \
	: spawn /bin/echo `/bin/date` access denied>>/var/log/sshd.log \
	: deny `

Her option field’ın önünde ters eğik çizginin (\) kullanılması, kuralın uzunluktan dolayı başarısız olmasını önler.
Bu örnek kural, example.com domain'deki bir host'dan SSH daexxmon (sshd) ile bağlantı kurulmaya çalışılırsa, girişimi özel bir log dosyasına eklemek için echo komutunu çalıştıracağını ve bağlantıyı reddettiğini belirtir. İsteğe bağlı deny yönergesi kullanıldığından, bu satır, hosts.allow dosyasında görünse bile erişimi reddeder.

#### 	Wildcardlar
•	Wildcardlar, TCP Wrappers'ın daemons veya host gruplarıyla daha kolay eşleşmesini sağlar. Aşağıdaki wildcards mevcuttur:
•	ALL - Her şeyle eşleşir. Hem daemon listesi hem de client listesi için kullanılabilir.
•	LOCAL - Localhost gibi nokta (.) İçermeyen herhangi bir host ile eşleşir.
•	KNOWN - Hostname ve host adreslerinin bilindiği veya kullanıcının bilindiği herhangi bir host ile eşleşir.
•	UNKNOWN - Hostname veya host adresinin bilinmediği veya kullanıcının bilinmediği herhangi bir host ile eşleşir.
•	PARANOID - Host adını elde etmek için kaynak IP adresinde bir reverse DNS araması yapılır. Ardından IP adresini çözmek için bir DNS araması gerçekleştirilir. İki IP adresi eşleşmezse, bağlantı kesilir ve log'lar güncellenir.


##### 	Patternlar
Patterns, client hosts gruplarını daha kesin olarak belirtmek için erişim kurallarının client field’da kullanılabilir. Aşağıda, client field'deki girişler için yaygın olarak kullanılan pattern'ların bir listesi verilmiştir:
•	Nokta (.) İle başlayan Hostname - Bir hostname'nin başına nokta koymak, adın listelenen bileşenlerini paylaşan tüm host'lerle eşleştirir. Aşağıdaki örnek, example.com domain içindeki herhangi bir host için geçerlidir:
ALL : .example.com
•	Nokta (.) İle biten IP adresi - Bir IP adresinin sonuna nokta koymak, bir IP adresinin initial numeric gruplarını paylaşan tüm host'larla eşleştirir. Aşağıdaki örnek, 192.168.x.x network içindeki herhangi bir host için geçerlidir:
ALL : 192.168.
•	IP adresi/netmask pair - Netmask ifadeleri, belirli bir IP adresi grubuna erişimi kontrol etmek için bir pattern olarak da kullanılabilir. Aşağıdaki örnek, adres aralığı 192.168.0.0 - 192.168.1.255 olan tüm host'lar için geçerlidir:
ALL : 192.168.0.0/255.255.254.0
•	[IPv6 adresi]/prefixlen pair - [net]/prefixlen pair’lar belirli bir IPv6 adres grubuna erişimi kontrol etmek için bir model olarak da kullanılabilir. Aşağıdaki örnek, adres aralığı 3ffe: 505: 2: 1 ::  -  3ffe: 505: 2: 1: ffff: ffff: ffff: ffff olan tüm hoxst'ler için geçerlidir:
ALL : [3ffe:505:2:1::]/64
•	Yıldız işareti (*) - Yıldız işaretleri, diğer pattern türlerini içeren karmaşık bir listede karıştırılmadıkları sürece, tüm hostname’leri veya IP adres gruplarını eşleştirmek için kullanılabilir. Aşağıdaki örnek, example.com domain içindeki herhangi bir hoxt için geçerlidir:
ALL : *.example.com
•	Eğik çizgi (/) - Bir client listesi eğik çizgiyle başlıyorsa, dosya adı olarak değerlendirilir. Bu, çok sayıda host'u belirten kurallar gerekliyse kullanışlıdır. Aşağıdaki örnek, tüm Telnet bağlantıları için TCP Wrappers'ı /etc/telnet.hosts dosyasına başvurur:
in.telnetd : /etc/telnet.hosts


#### 	Portmap ve TCP Wrapper’lar
Portmap'in TCP Wrappers uygulaması host aramalarını desteklemez, bu da portmap'in host'ları tanımlamak için hostname’leri kullanamayacağı anlamına gelir. Sonuç olarak, hosts.allow veya hosts.deny'deki portmap için erişim kontrol kuralları, host'ları belirtmek için IP adreslerini veya ALL anahtar kelimesini kullanmalıdır.
Portmap erişim kontrol kurallarında yapılan değişiklikler hemen etkili olmayabilir. Portmap hizmetini yeniden başlatmanız gerekebilir.
NIS ve NFS gibi yaygın olarak kullanılan hizmetler, çalışmak için portmap'e bağlıdır, bu nedenle bu sınırlamalara dikkat edilmelidir.


#### 	Operator’ler
Şu anda, erişim kontrol kuralları bir operatör olan EXCEPT'i kabul etmektedir. Hem daemon listesinde hem de bir kuralın client listesinde kullanılabilir. hosts.allow dosyasından alınan aşağıdaki örnekte, tüm example.com host'ların attacker.example.com dışındaki tüm hizmetlere bağlanmasına izin verilir:
ALL : .example.com EXCEPT attacker.example.com
Bir hosts.allow dosyasından başka bir örnekte, 192.168.0.x network'deki client'ler FTP dışındaki tüm hizmetleri kullanabilir:
ALL EXCEPT vsftpd : 192.168.0.
