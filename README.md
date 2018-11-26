# Django Yayın Ortamı Yapılandırması

## Temel Güvenlik Duvarı Ayarları

1. Güvenlik duvarında SSH'a izin verme
```
sudo ufw allow OpenSSH
```

2. Güvenlik duvarı kurallarını aktif etme.
```
sudo ufw enable
```

3. Güvenlik duvarı kurallarını listeleme
```
sudo ufw status
```

## Paketleri Güncelleme
```
sudo apt-get update
```

## Python3, PostgreSQL ve Nginx bileşenlerinin yüklenmesi
```
sudo apt-get install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx
```

```
python3 -V
pip3 -V
```

## PostgreSQL Yapılandırması

1. İnteraktif PostgreSQL komut istemcisine bağlanma
```
sudo -u postgres psql
```

2. Projenin kullanacağı veritabanını oluşturma
```
CREATE DATABASE proje_adi;
```

3. Projenin kullanacağı veritabanı kullanıcısını oluşturma
```
CREATE USER kullanici_adi WITH PASSWORD 'parola123';
```

4. Varsayılan karakter kodlaması (default character encoding), işlem (transaction) ve bölgesel zaman ile ilgili ayarların yapılması
```
ALTER ROLE kullanici_adi SET client_encoding TO 'utf8';
ALTER ROLE kullanici_adi SET default_transaction_isolation TO 'read committed';
ALTER ROLE kullanici_adi SET timezone TO 'Europe/Istanbul';
```

5. Oluşturulan kullanıcıyı veritabanına yönetici erişimi izni verme
```
GRANT ALL PRIVILEGES ON DATABASE proje_adi TO kullanici_adi;
```

6. PostgreSQL komut istemcisinden çıkmak
```
\q
```

## Proje için Python Sanal Ortamın (Virtual Environment) ve Bağımlılıkların Yüklenmesi

1. Python3 için pip güncellemesi ve virtualenv paketinin kurulması
```
sudo -H pip3 install --upgrade pip
sudo -H pip3 install virtualenv
```

Not: "locale.Error: unsupported locale setting" hatası için çözüm:
```
export LC_ALL="en_US.UTF-8"
```

2. Örnek bir Django projesi GitHub'dan kopyalanıyor.
```
git clone https://github.com/serdardeniz/eventhub.git
```

3. Proje dizinine geçme ve venv adlı Sanal Python Ortamının oluşturulması
```
cd eventhub
virtualenv venv
```

4. Sanal Ortamın aktif edilmesi
```
source venv/bin/activate
```

5. Proje bağımlılıklarının yüklenmesi
```
pip install -r requirements.txt
```

6. Gunicorn ve PostgreSQL adaptörünün yüklenmesi
```
pip install gunicorn psycopg2
```

Not: Projenin settings.py dosyasının dizin hiyerarşisi aşağıdaki gibidir:
```
/home/django/eventhub/
		    /accounts/		
		    /event/
		    /eventhub/
		   	     /__init__.py	
			     /settings.py
			     /urls.py
			     /wsgi.py
		    /notification/
		    /static/
		    /templates/
		    . . .
```

## Projeyi Django'nun Geliştirme Sunucusuyla Test Etme

1. Test edilecek 8000 portunu aktif etme

```
sudo ufw allow 8000
```

2. Geliştirme sunucusunu çalıştırma
```
python manage.py runserver 0.0.0.0:8000
# 0.0.0.0 -> sunucunun IP adresi. (sunucu kendi IP adresini bildiğinden 0.0.0.0 yazılabilir)
```

3. Tarayıcıdan test etme
```
http://alan_adi_veya_IP:8000
```

## Gunicorn

1. Projeyi Gunicorn ile Çalıştırarak Gunicorn'u Test Etmek
```
# Sanal ortam aktif olmalı ve manage.py dizininde çalıştırılmalı. (eventhub.wsgi -> proje_adı.wsgi)
gunicorn --bind 0.0.0.0:8000 eventhub.wsgi
```

Tarayıcıdan test etme: http://alan_adi_veya_IP:8000
Gunicorn sunucusunu durduma: `CTRL + C`

2. Gunicorn `systemd` Servis Dosyası Oluşturma
```
# Sanal ortamdan çıkılmalı
deactivate
# Servis dosyası oluşturma
sudo nano /etc/systemd/system/gunicorn.service
```

gunicorn.service dosya içeriği şu şekilde olmalıdır:

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=django
Group=www-data
WorkingDirectory=/home/django/eventhub
ExecStart=/home/django/eventhub/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/django/eventhub/eventhub.sock eventhub.wsgi:application

[Install]
WantedBy=multi-user.target
```

Not: Sanal ortamın adı `venv`, projenin adı `eventhub`, kullanıcı adı `django` olarak varsayılmıştır.

Not 2: Dosyayı kaydetmek için `CTRL + X` yaptıktan sonra `y` harfine basıp, `enter` tuşuna basılmalı.

3. Gunicorn servisini aktif etme:
```
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

4. Gunicorn servisinin doğru bir şekilde çalışıp çalışmadığını test etme:
```
sudo systemctl status gunicorn
# Yeşil renkte "active (running)" yazıyorsa başarılı bir şekilde çalışıyor demektir.
```

```
ls /home/baris/eventhub
# ls çıktısında "proje_adı.sock" adlı (.sock) uzantılı bir soket dosyası görülüyorsa gunicorn başarılı bir şekilde yapılandırılmıştır.
```

## Nginx Yapılandırması

1. Nginx sunucu bloğu açma:
```
sudo nano /etc/nginx/sites-available/eventhub
```

Dosyanın içinde bulunması gerekenler:
```
server {
    listen 80;
    server_name alan_adi_veya_IP;
    root /home/django/eventhub; # Projenin kök dizini

    location /static/ {
    }

    location /media/ {
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/django/eventhub/eventhub.sock;  # Projenin kök dizinindeki 'proje_adı.sock' dosyası
    }
}
```

Not: Dosyayı kaydetmek için `CTRL + X` yaptıktan sonra `y` harfine basıp, `enter` tuşuna basılmalı.

2. Nginx dosyasını aktif etmek için dosyayı sites-enabled dizinine link olarak verme
```
sudo ln -s /etc/nginx/sites-available/eventhub /etc/nginx/sites-enabled
```

3. Nginx yapılandırma dosyalarında syntax hatası olup olmadığını kontrol etme
```
sudo nginx -t
```

4. Hata yoksa Nginx sunucusunu yeniden başlatma
```
sudo systemctl restart ngin
```

5. Kullanılmayacak olan 8000 portunu kapatma ve Nginx kurallarını aktif etme
```
sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full'
```
