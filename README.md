# Dovecot-Fail2ban-Security

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

## Katkı

Katkılarınızı bekliyoruz! Herhangi bir iyileştirme veya öneri için lütfen bir sorun açın veya bir çekme isteği gönderin.














