# Dovecot-Fail2ban-Security

![image](https://github.com/ugurcomptech/Dovecot-Fail2ban-Security/assets/133202238/7efad4dc-971d-4990-84ab-68a865b5c9bf)


Bu proje, IMAP ve POP3 hizmetlerine yönelik kaba kuvvet saldırılarına karşı koruma sağlamak için Dovecot'u Fail2Ban ile entegre ederek bir e-posta sunucusunun güvenliğini sağlamayı amaçlamaktadır.


## Özellikler

- Dovecot IMAP ve POP3 hizmetlerini kaba kuvvet saldırılarına karşı korur
- Fail2Ban kullanarak IP yasağı sağlar
- Yapılandırılabilir yeniden deneme sınırları ve yasaklama süreleri


## Önkoşullar

- Ubuntu veya herhangi bir Debian tabanlı sistem
- Dovecot
- Fail2Ban

## Kurulum

### Fail2ban'ı yükleyin

```sh
sudo apt-get update
sudo apt-get install fail2ban
```


## Konfigürasyon

### Dovecot Konfigürasyon

Dovecot hizmet limitlerini düzenleyin:

```
sudo nano /etc/dovecot/conf.d/10-master.conf
```

```
service imap {
  service_count = 1
  process_limit = 100
  client_limit = 10
}
```


### Dovecot için Fail2Ban Yapılandırması

Dovecot için bir filtre oluşturun:

```
sudo nano /etc/fail2ban/filter.d/dovecot.conf
```

Aşağıdaki içeriği ekleyin:

```
[INCLUDES]
before = common.conf

[Definition]
_daemon = (?:dovecot)
failregex = (?:Authentication failure.*rip=<HOST>)|(?:Aborted login.*rip=<HOST>)|(?:Disconnected.*no auth attempts.*rip=<HOST>)
ignoreregex =
```


Dovecot için jail dosyasını yapılandırın:

```
sudo nano /etc/fail2ban/jail.local
```

Aşağıdaki içeriği ekleyin:

```
[dovecot]
enabled = true
port = imaps,pop3,pop3s
filter = dovecot
logpath = /var/log/mail.log
maxretry = 2
findtime = 600
bantime = 3600
```

## Servisleri Yeniden Başlat

Dovecot ve Fail2Ban'ı yapılandırdıktan sonra değişiklikleri uygulamak için her iki hizmeti de yeniden başlatın:

```
sudo systemctl restart dovecot
sudo systemctl restart fail2ban
```


## Testler

Bir e-posta istemcisi kullanarak hatalı oturum açma denemeleri gerçekleştirin ve günlükleri inceleyin:

```
sudo tail -f /var/log/mail.log
sudo tail -f /var/log/fail2ban.log
```

Fail2Ban'ın durumunu ve yasaklı IP adreslerini kontrol edin:

```
sudo fail2ban-client status dovecot
```

## Webmail Üzerinde Gelen İstekleri Engellemek

SnappyMail için bir Fail2Ban filtre dosyası oluşturun. Bu dosya, fazla istekleri algılamak için kullanılır.

```
sudo nano /etc/fail2ban/filter.d/snappymail.conf
```

Aşağıdaki içeriği ekleyin:

```
[Definition]
failregex = ^<HOST> .*POST /webmail/\?r=api/login.*
ignoreregex =
```

Bu yapılandırma, belirli bir süre içinde belirli sayıda başarısız giriş denemesinden sonra IP adreslerini engeller.


```
sudo nano /etc/fail2ban/jail.local
```

Aşağıdaki içeriği ekleyin:

```
[snappymail]
enabled = true
port = http,https
filter = snappymail
logpath = /usr/local/lsws/logs/access.log
maxretry = 5
findtime = 600
bantime = 3600
```

### LiteSpeed Web Sunucusu Yapılandırması

LiteSpeed web sunucusu için, belirli IP adreslerinden gelen fazla istekleri sınırlayarak sunucuyu koruyabilirsiniz.

```
sudo nano /usr/local/lsws/conf/httpd_config.conf
```

Aşağıdaki içeirği ekleyin:

```
<IfModule Litespeed>
    SetEnvIf Request_URI "^/webmail/\?r=api/login" rate-limit
    <Location "/webmail">
        LimitReqZone rate-limit zone=snappymail:10m rate=5r/m
    </Location>
</IfModule>
```


### SnappyMail Konfigürasyon Dosyasını Düzenleme

```
sudo nano /usr/local/lscp/cyberpanel/rainloop/data/_data_/_default_/configs/application.ini
```

Aşağıdaki içeirği ekleyin:

```
[security]
; Maksimum giriş denemesi sayısı
login_attempt_limit = 5

; Giriş denemesi sınırına ulaşıldığında bekleme süresi (saniye cinsinden)
login_attempt_delay = 60
```

### Servisleri Yeniden Başlatma

```
sudo systemctl restart fail2ban
sudo systemctl restart lsws
```

Webmail üzerinden gelen istekler sunucunun  `127.0.0.1` Local adresinden gelir. O yüzden fail2ban bunu es geçer. Bunu es geçmemesi için `jail.local` dosyasına `ignoreself = false` komutunu ekleyiniz.


## Test Etmek

Aşağıdaki komut ile loglara bakarak test sağlayabilirsiniz.

```
tail -f /var/log/fail2ban.log
```


## IP Whitelist

İstemiş olduğunuz IP adresinden gelen istekleri engellememek için bir kural oluşturabilirsiniz. 

Fail2Ban'ın jail.local dosyasını düzenleyerek ignoreip parametresini ekleyelim:

```
sudo nano /etc/fail2ban/jail.local
```


Dosyanın içine aşağıdaki gibi ignoreip parametresini ekleyin:


```
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 192.168.1.100   ; Engellenmeyecek IP adreslerini buraya ekleyiniz.
```

Bu örnekte, ignoreip parametresi altında 192.168.1.100 IP adresi belirtilmiştir. Böylece, fail2ban bu IP adresini engellemeyecektir.

Değişiklikleri uydukladıktan sonra servisimizi yeniden başlatalım.

```
sudo systemctl restart fail2ban
```


Eklemiş olduğum Ip adresini test ettiğimde engellemedğini görüntülemiş olldum.

```
2024-06-26 22:54:36,621 fail2ban.filter         [113300]: INFO    [dovecot] Ignore 88.241.72.246 by ip
2024-06-26 22:54:52,648 fail2ban.filter         [113300]: INFO    [dovecot] Ignore 88.241.72.246 by ip
```

## Manuel BAN / UNBAN

Manuel olarak IP adresi banlamak istiyor iseniz veya IP banı kaldırmak istiyor iseniz aşağıdaki adımlar izleyiniz.

`sudo fail2ban-client status dovecot` komutu ile **dovecot** Jailinde hangi IP adresleri yasaklanmış onları görüntülüyoruz.

```
root@ubuntu:~# sudo fail2ban-client status dovecot
Status for the jail: dovecot
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     2
|  `- File list:        /var/log/mail.log
`- Actions
   |- Currently banned: 1
   |- Total banned:     3
   `- Banned IP list:   212.51.4.78
```

`sudo fail2ban-client set dovecot unbanip <IP_Adresi>` komutunu yazarak istemiş olduğunuz IP adresinin yasaklanmasını kaldırabilirsiniz.

```
root@ubuntu:~# sudo fail2ban-client set dovecot unbanip 212.51.4.78
1
root@ubuntu:~# sudo fail2ban-client status dovecot
Status for the jail: dovecot
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     2
|  `- File list:        /var/log/mail.log
`- Actions
   |- Currently banned: 0
   |- Total banned:     3
   `- Banned IP list:
```

`sudo fail2ban-client set dovecot banip <IP_Adresi>` komutunu yasarak istemiş olduğunuz IP adresini yasaklayabilirsiniz.

```
root@ubuntu:~# sudo fail2ban-client set dovecot banip 212.51.4.78
1
root@ubuntu:~# sudo fail2ban-client status dovecot
Status for the jail: dovecot
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     2
|  `- File list:        /var/log/mail.log
`- Actions
   |- Currently banned: 1
   |- Total banned:     5
   `- Banned IP list:   212.51.4.78
root@ubuntu:~# 
```





## Katkı

Katkılarınızı bekliyoruz! Herhangi bir iyileştirme veya öneri için lütfen bir sorun açın veya bir çekme isteği gönderin.



## Lisans

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Bu projeyi [MIT Lisansı](https://opensource.org/licenses/MIT) altında lisansladık. Lisansın tam açıklamasını burada bulabilirsiniz.











