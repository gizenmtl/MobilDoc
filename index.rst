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

**COMPOSER PHP PAKET YÖNETİCİSİ**
=================================

####

**Adım 1:** Yüklemeye başlamadan önce bağımlılıklarımızı ve paketlerimizi güncelleyelim.

$ sudo apt-get update

**Adım 2:** $ sudo apt-get install curl php-cli php-mbstring git unzip

**Adım 3:** Home klasörümüze gidip composer'ı curl ile yükleyeceğiz.

$ cd ~

$ curl -sS https://getcomposer.org/installer -o composer-setup.php

php -r "if (hash_file('SHA384', 'composer-setup.php') === '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

Installer verified çıktısı almanız gerekir.


Şimdi global olarak composer'ı yükleyeceğiz.

sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer

Artık composer'ımız /usr/local/bin klasörümüze yüklendi.

Yüklememizi test etmek için

$ composer yazalım ve artık composer'ımız da yüklendi.


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



Artık nginx sunucumuzun da kurulumu tamamlandığına göre, postgresql'e geçebiliriz.


**POSTGRESQL KURULUMU**
=======================

####


**Adım 1:** https://www.postgresql.org/ sitesine girip Download bölümüne tıklıyoruz.

**Adım 2:** Karşımıza çıkan sayfadan kullandığımız Linux işletim sisteminin hangi dağıtımı olduğunu seçtikten sonra
hangi versiyonunu kullandığımızı da seçmeliyiz.

**Adım 3:** /etc/apt/sources.list.d uzantısının içine pgdg.list dosyası oluşturuyoruz (eğer yoksa).
Dosyamızı oluşturduktan sonra, içerisine

deb http://apt.postgresql.org/pub/repos/apt/ zesty-pgdg main yazıyoruz.

**Adım 4:** Ardından paket listelerini güncellemek ve giriş anahtarını entegre etmek için terminalimize,

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
  sudo apt-key add -
sudo apt-get update

yazıyoruz.

**Adım 5:** Konfigürasyonlar ..



**Node.js-NPM KURULUMU**
========================

####

**Adım 1:** İlk olarak paketi güncellemeyi alışkanlık haline getirelim.

$ sudo apt-get update

**Adım 2:** Şimdi Node.js yükleyelim.

$ sudo apt-get install nodejs

**Adım 3:** Node.js paket yöneticisi npm'i aşağıdaki komut ile kurabiliriz.

$ sudo apt-get install npm

**PhpStorm Kurulumu**
=====================

####

**Adım 1:** https://www.jetbrains.com/ sitesinden öğrenci hesabınız ile giriş yaptığınızda, JetBrains'in bütün IDE'lerini professionel olarak kullanabilirsiniz.

**Adım 2:** Öğrenci mail adresiniz ile üye olduktan sonra, PhpStorm'u indirip size gelen mail ile entegre etmelisiniz.

**Adım 3:** Yükleme tamamlandıktan sonra projenizi yazmaya başlayabilirsiniz.

**Google Chrome Kurulumu**
==========================

####

**Adım 1:** İlk olarak anahtarımızı eklemeliyiz.

$ wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

**Adım 2:** Repo ile entegre ediyoruz.

$ echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' | sudo tee /etc/apt/sources.list.d/google-chrome.list

**Adım 3:** Artık paketleri yükleyebiliriz.

$ sudo apt-get update

$ sudo apt-get install google-chrome-stable

**Adım 4:** Artık Google Chrome browserımız kullanıma hazır.

**Git Kurulumu ve ssh Bağlantısı**
==================================

####

**Adım 1:** $ sudo apt-get update

**Adım 2:** Paketlerimizi güncellediğimize göre artık Git 'i yükleyebiliriz.

$ sudo apt-get install git

**Adım 3:** Şimdi Git hesabımızla bilgisayarımızı eşleştirmek için, SSH bağlantısını oluşturmaya başlayacağız.

    $ git config --global user.name "Your Name"
    $ git config --global user.email "youremail@domain.com"

**Adım 4:** SSH key'imize ulaşmak için,

$ cd ~/.ssh

$ cat id_rsa.pub

yazdıktan sonra karşımıza çıkan key'i GitHub/GitLab hesabınıza girip Ayarlar > SSH and GPG keys
yolunu izledikten sonra New SSH Key butonuna basıp, id_rsa.pub dosyasından kopyaladığımız key'imizi buraya yapıştırıyoruz.

Ve artık bilgisayarımızla Git hesabımız eş zamanlı bir şekilde çalışabilir.




