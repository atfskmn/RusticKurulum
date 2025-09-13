# Restic Kurulum

# 1. Sistem Güncellemeleri
sudo apt update && sudo apt upgrade -y
# 2. Restic Kurulumu (Ubuntu)
## Restic indir
wget https://github.com/restic/restic/releases/download/v0.16.2/restic_0.16.2_linux_amd64.bz2

## Dosyayı çıkar
bzip2 -d restic_0.16.2_linux_amd64.bz2

## Çalıştırma izni ver
chmod +x restic_0.16.2_linux_amd64

## Sistem PATH'ine taşı
sudo mv restic_0.16.2_linux_amd64 /usr/local/bin/restic

## Kurulumu doğrula
restic version

# 3. MySQL Kurulumu

## Not: Eğer MySQL zaten kurulu ise bu adımları atlayabilirsiniz

## MySQL server kur
sudo apt install mysql-server -y

## MySQL servisini başlat
sudo systemctl start mysql
sudo systemctl enable mysql

## Güvenlik ayarlarını yap
sudo mysql_secure_installation

### 3.1. MySQL Bağlantı Testi

## MySQL bağlantısını test et
mysql -u root -p -e "SHOW DATABASES;"

# 4. MySQL Yapılandırması

## MySQL'e bağlan
sudo mysql -u root -p

## Örnek veritabanı oluştur
CREATE DATABASE company;
USE company;

CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    department VARCHAR(50)
);

INSERT INTO employees (name, email, department) VALUES
('Ahmet Yılmaz', 'ahmet@company.com', 'IT'),
('Ayşe Demir', 'ayse@company.com', 'HR'),
('Mehmet Kaya', 'mehmet@company.com', 'Finance');

### 4.1. Veritabanı Doğrulama

## Veritabanının oluşturulduğunu kontrol et
mysql -u root -p -e "SHOW DATABASES;"

# 5. Restic Reposu Oluşturma

## Backup dizini oluştur
sudo mkdir -p /backup/restic-repo

## Klasör sahipliğini ayarla
sudo chown -R $USER:$USER /backup/restic-repo

## Kalıcı environment variables ekle
echo 'export RESTIC_REPOSITORY=/backup/restic-repo' >> ~/.bashrc
echo 'export RESTIC_PASSWORD="guclu_sifre_123"' >> ~/.bashrc
echo 'export RESTIC_COMPRESSION=auto' >> ~/.bashrc

## Değişkenleri yükle
source ~/.bashrc

# 5.1. Environment Variables Kontrolü
## Environment variables'ların doğru ayarlandığını kontrol et
echo $RESTIC_REPOSITORY
echo $RESTIC_PASSWORD

# 6. Repository Initialize Yapma

## Repoyu başlat
restic init

## Başarı mesajı kontrol et
echo "Repository oluşturuldu: $(restic snapshots 2>/dev/null | head -n 1)"

# 7. Backup Script Oluşturma
sudo nano /usr/local/bin/mysql-backup.sh

### 7.1. Script İçeriği:

#!/bin/bash

### MySQL backup script
export RESTIC_REPOSITORY=/backup/restic-repo
export RESTIC_PASSWORD="guclu_sifre_123"

TIMESTAMP=$(date +"%Y-%m-%d-%H-%M")
BACKUP_FILE="/tmp/mysql-backup-$TIMESTAMP.sql"

### MySQL dump al
echo "$(date) - MySQL yedekleme başlıyor..."
mysqldump --all-databases > $BACKUP_FILE

### Restic ile yedekle
echo "$(date) - Restic'e yükleniyor..."
restic backup $BACKUP_FILE

### Retention policy uygula
echo "$(date) - Eski yedekler temizleniyor..."
restic forget --keep-hourly 24 --keep-daily 7 --keep-weekly 4 --keep-monthly 3 --keep-yearly 2 --prune

### Temizlik
rm -f $BACKUP_FILE
echo "$(date) - Yedekleme tamamlandı!"


## 7.2. Script İzinleri:

sudo chmod +x /usr/local/bin/mysql-backup.sh
sudo chown root:root /usr/local/bin/mysql-backup.sh
## 7.2.1. Script Testi

## Script'i test et
sudo /usr/local/bin/mysql-backup.sh

# 8. Cron Job Ekleme

sudo crontab -e
### 8.1. Cron Satırları:

### Her gün gece 02:00'de yedek al
0 2 * * * /usr/local/bin/mysql-backup.sh >> /var/log/mysql-backup.log 2>&1

### Her saat başı yedek al (opsiyonel)
# 0 * * * * /usr/local/bin/mysql-backup.sh >> /var/log/mysql-backup-hourly.log 2>&1
## 8.2. Cron Kontrolü
### Cron job'ları listele
sudo crontab -l

## Cron log'larını kontrol et
sudo tail -f /var/log/syslog | grep CRON
# 9. Detay Görüntüleme
## 9.1. Snapshot Komutları:
bash
### Tüm snapshot'ları listele
restic snapshots

# JSON formatında listele
restic snapshots --json

# En son 5 snapshot'ı göster
restic snapshots --last 5

# Tarihe göre filtrele
restic snapshots --time="2025-09-14"

# Belirli bir dosya yolundaki snapshot'lar
restic snapshots --path "/tmp/mysql-backup"
### 9.2. Repository İstatistikleri
bash
# Repository boyutu
restic stats

# Repository bütünlük kontrolü
restic check

# 10. Tebrikler!
### Artık yedek alınmaya başlandı. Sisteminiz otomatik olarak MySQL veritabanlarınızı yedekleyecek ve eski yedekleri temizleyecek.






















