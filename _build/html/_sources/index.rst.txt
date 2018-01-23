.. demo documentation master file, created by
   sphinx-quickstart on Mon Jan 22 17:08:00 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

**VMWARE WorkStation Pro Sanal Makine Kurulumu**
================================================

####

.. toctree::
   :maxdepth: 2
   :caption: Contents:

VMware WorkStation programını kurduktan sonra ihtiyacımıza göre sanal bir makine oluşturuyoruz.
Programı çalıştırıyoruz>File>New Virtual Machine butonunu kullanarak yeni bir sanal makine kuruyoruz.

**Sanal Makinemize Ubuntu Kurulumu**
====================================

####

Karşımıza çıkan ekranda Typical sekmesini işaretliyor ve next diyoruz.Dilersek burada custom seçeneğini seçip kendi konfigürasyonumuzu belirtebiliriz. Ardından kurulum için kaynak dosyamızı seçiyor ve devam ediyoruz. Kaynak dosyasını (.iso uzantılı dosya) zaten indirmiştik.
Daha sonra bize username ve password bilgisi girmemizi söylüyor. Burada istediğiniz kullanıcı adı ve paralosını girip, next tuşuna basıyoruz. Daha sonraki adımda VM adımızı soruyor varsayılan olarak bırakıp next diyebiliriz ya da dilediğimiz bir ismi burada tanımalayabiliriz. Bu isim VM içerisinde gözükecek olan isimdir.

Bilgisayarınızın harddisk boyutuna göre 20 gb ve üstü alan seçip ardından split seçeneğini seçiyoruz. Biz 40gblık bir alan seçtiğimizden hız olarak bir fark olmayacaktır. Ancak dilerseniz yine single file seçeneğini seçebilirsiniz.
Ve finish sekmesine basıp VM ayarlarını bitirip artık kuruluma geçiyoruz. Kurulum başladıktan sonra ise yazılım güncellemeleri otomatik olarak kontrol edilmeye başlar.
Ubuntu yüklendikten sonra, kullanıcı oluşturmamız için birkaç ufak bilgi soruyor ve kurulum devam ediyor.
Kurulum tamamlandıktan sonra sisteme login oluyoruz ve sanal makinemizdeki ubuntumuz artık kullanıma hazır :)



**PHP Kurulumu**
================

####

**Adım 1**: Ubuntu'yu SSH ile bağlıyoruz ve Ondrej’s PPA ile erişmesini sağlıyoruz.


$ sudo apt-get install software-properties-common

$ sudo add-apt-repository ppa:ondrej/php

$ sudo apt-get update




**Adım 2:** Install PHP 7.1

$ sudo apt-get install php7.1


**Adım 3:** PHP 7.1 bağımlılıkları ve Modüllerinin yüklenmesi

$ sudo apt-cache search php7.1


**Adım 4:** Yaygın kullanılan modüllerin yüklenmesi

$ sudo apt-get install php7.1 php7.1-cli php7.1-common php7.1-json php7.1-opcache php7.1-mysql php7.1-mbstring php7.1-mcrypt php7.1-zip php7.1-fpm


**Adım 5:** php.ini konfigürasyonları

Yüklemeler tamamlandıktan sonra /etc/php/7.1/cli/php.ini dosyasını açıp, konfigürasyonlarının tamamlanması gerekir.

$ sudo nano /etc/php/7.1/cli/php.ini ile dosyamızı açıyoruz.

cgi.fix_pathinfo=0 ile değiştiriyoruz.

Ardından PHP-FPM servisini yeniden başlatıyoruz.

$ sudo systemctl restart php7.1-fpm.service

Artık PHP konfigürasyonlarımız da tamamlandı.


**NGINX SERVER KURULUMU**
=========================

####


**Adım 1:** $ sudo apt-get install nginx

**Adım 2:** sites-available klasörünün içine local.example.com adlı bir dosya oluşturmalısınız.

$ sudo nano /etc/nginx/sites-available

$ touch local.example.com.conf


**Adım 3:** Domain ismine göre nginx sanal sunucusunu kurma

$ sudo nano /etc/nginx/sites-available/local.example.com.conf


**Adım 4:** Açılan dosyamızın içinde;

server {
        listen 80;
        server_name local.example.com;
        root /www/user_name/public;
        index index.php index.html index.htm;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            fastcgi_pass unix:/run/php/php7.1-fpm.sock;
            include snippets/fastcgi-php.conf;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }

        location ~ /\.ht {
                deny all;
        }
}


ile sites-avaible içerisinde oluşturduğumuz linki gerekli yerleri değiştirerek sites-enabled içerisine atmalıyız.

**Adım 5:** Şimdi sites-enabled içerisine kopyalamak için;


$ ln -s /etc/nginx/sites-avaible/local.example.com /etc/nginx/sites-enabled/local.example.com


**Adım 5:** Nginx konfigürasyonu test etme

$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

**Adım 6:** Eğer "syntax is ok" cevabı aldıysanız, artık Nginx server'ı restart edebilirsiniz

$ sudo systemctl restart nginx.service

**Adım 7:** Sistem boot'u üzerinden Nginx ve PHP-FPM'e ulaşma

$ sudo systemctl enable nginx.service

$ sudo systemctl enable php7.1-fpm.service

























Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
