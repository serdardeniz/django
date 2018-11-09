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
git clone https://github.com/barissaslan/eventhub.git
```

3. Proje dizinine geçme ve venv adlı Sanal Python Ortamının oluşturulması
```
cd eventhub
virtualenv venv
```
