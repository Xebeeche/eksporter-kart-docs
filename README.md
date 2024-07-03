# Eksporter KART

Aplikacja przeznaczona jest dla Okręgowych Komisji Egzaminacyjnych, które chciałyby zautomatyzować proces eksportu kart i zdających, do aplikacji ONE, dla testowych lub szkoleniowych sesji oceniania.
Oparta jest o framework PHP [Laravel 11](https://laravel.com).

## Spis treści

1. [Eksporter KART](#eksporter-kart)
2. [Screenshots](#screenshots)
3. [Możliwości](#możliwości)
4. [Opis działania aplikacji Eksporter KART - tutaj](#opis-działania-aplikacji-eksporter-kart---tutaj)
5. [Instalacja automatyczna za pomocą Packera](#instalacja-automatyczna-za-pomocą-packera)
   - [Czynności wstępne](#czynności-wstępne)
   - [Konfiguracja sieci](#konfiguracja-sieci)
   - [Użytkownicy](#użytkownicy)
   - [Zmienne](#zmienne)
   - [Skrypt eksporter-kart_setup.sh](#skrypt-eksporter-kart_setupsh)
   - [Skrypt cleanup.sh](#skrypt-cleanupsh)
   - [Uruchamianie Packera](#uruchamianie-packera)
6. [Instrukcja ręcznej instalacji aplikacji Eksporter KART na Linuxie](#instrukcja-ręcznej-instalacji-aplikacji-eksporter-kart-na-linuxie)
   - [Wymagania systemowe](#wymagania-systemowe)
   - [1. Aktualizacja systemu i instalacja pakietów](#1-aktualizacja-systemu-i-instalacja-pakietów)
   - [2. Dodawanie wymaganych repozytoriów apt](#2-dodawanie-wymaganych-repozytoriów-apt)
   - [3. Instalacja sqlsrv driver](#3-instalacja-sqlsrv-driver)
   - [4. Instalacja rozszerzenia smbclient](#4-instalacja-rozszerzenia-smbclient)
   - [5. Tworzenie bazy danych i użytkownika](#5-tworzenie-bazy-danych-i-użytkownika)
   - [6. Instalacja aplikacji z prywatnego repozytorium GitHub](#6-instalacja-aplikacji-z-prywatnego-repozytorium-github)
   - [7. Przykładowa konfiguracja Nginx](#7-przykładowa-konfiguracja-nginx)
   - [8. Konfiguracja Supervisor](#8-konfiguracja-supervisor)
7. [Czynności poinstalacyjne](#czynności-poinstalacyjne)
   - [Zmiana haseł użytkownika root i packer](#zmiana-haseł-użytkownika-root-i-packer)
   - [Dostęp do systemu i generowanie kluczy SSH](#dostęp-do-systemu-i-generowanie-kluczy-ssh)
8. [Licencja](#licencja)

## Screenshots

[Repozytoria kart odpowiedzi](Screenshots/repozytoria.png)

[Nowe repozytorium]((Screenshots)/repozytorium_nowe.png)

[Edycja repozytorium](Screenshots/akcje_masowe.png)

[Eksporty kart odpowiedzi]([Screenshots](Screenshots)/eksporty.png)

[Nowy eksport](Screenshots/eksport_przygotowanie.png)

[Widok eksportu](Screenshots/eksporty.png)

[Uruchomienie eksportu](Screenshots/eksport_uruchomienie.png)

[Edycja użytkownika](Screenshots/uzytkownik_edycja.png)

## Możliwości

- Tworzenie repozytoriów kart.
- Tworzenie eksportów do ONE z dowolnego repozytorium kart.
- Dodawanie dostosowań dla importowanych do ONE zdających.
- Rozbudowany system powiadomień.
- Responsywny i nowoczesny interfejs.
- Obsługa motywów.
- Zarządzanie użytkownikami.
- Zarządzanie profilem użytkownika.
- Weryfikacja dwuetapowa użytkownika.
- Prosty deployment.

## [Opis działania aplikacji Eksporter KART - tutaj](OPIS.md)

## Instalacja automatyczna za pomocą Packera

Instalacja aplikacji Eksporter KART została zautomatyzowana za pomocą Packera od Hashicorp i umożliwia utworzenie maszyny wirtualnej VirtualBox (konieczna uprzednia instalacja [VirtualBox](https://www.oracle.com/pl/virtualization/technologies/vm/downloads/virtualbox-downloads.html))
albo instalację na Proxmox Virtual Environment. Proces tworzenia maszyny wirtualnej/szablonu trwa około 9-10 minut.

- [Packer - instalacja](https://developer.hashicorp.com/packer/install)

  Debian/Ubuntu

  ```sh
  wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
  echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com bookworm main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
  sudo apt update && sudo apt install packer
  ```

  CentOS/RHEL

  ```sh
  sudo yum install -y yum-utils
  sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
  sudo yum -y install packer
  ```

  macOS

  ```sh
  brew tap hashicorp/tap
  brew install hashicorp/tap/packer
  ```

### Czynności wstępne

Przed uruchomieniem procesu Packer, wykonaj poniższe kroki:

1. Sklonuj repozytorium z plikami Packer i przejdź do katalogu:

   ```sh
   git clone https://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:x-oauth-basic@github.com/xebeeche/eksporter-kart-deploy
   cd eksporter-kart-deploy
   ```

2. Inicjalizacja Packer w celu pobrania wymaganych pluginów:

   ```sh
   packer init eksporter-kart.pkr.hcl
   ```

3. Jeśli używasz Proxmox VE, pobierz obraz ISO Debiana na serwer PVE (adres URL podany w zmiennej `iso`):

   ```sh
   wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.5.0-amd64-netinst.iso
   ```

   Następnie przenieś pobrany plik do lokalnego storage w katalogu `/var/lib/vz/template/iso/`:

   ```sh
   mv debian-12.5.0-amd64-netinst.iso /var/lib/vz/template/iso/
   ```

### Konfiguracja sieci

  W pliku `http/static_preseed.cfg` znajdują się ustawienia automatycznej instalacji systemu Debian, w tym konfiguracji sieciowej. Domyślnie, dla Proxmox VE używany jest plik skonfigurowany na używanie statycznego adresu IP, zaś dla VirtualBox, który używa NAT, domyślnie korzysta z DHCP.

  Proxmox - stały adres IP

  W pliku `eksporter-kart.pkr.hcl` (linie 44-82) zdefiniowane są ręczne ustawienia sieci. Jeśli chcesz użyć DHCP, ustaw `disable_dhcp` na `disable_dhcp=false`.

### Użytkownicy

- Instalator ustawia hasło `root` na `r00tpass`.
- Tworzy użytkownika `packer` z hasłem `packer`.
- Domyślny użytkownik aplikacji: `admin` z hasłem `admin`.
- Użytkownikami systemowymi można zarządzać według potrzeb. Usuwanie czy zmiana hasła nie ma wpływu na działanie aplikacji.
- Możliwość dodawania i zarządzanie użytkownikami aplikacji.

### Zmienne

Kod HCL z pliku `eksporter-kart.pkr.hcl` na początku definiuje zmienne wykorzystywane w procesie budowania obrazów/szablonów maszyn wirtualnych. Zmienne istotne dla instalacji, zostały opisane poniżej.

Podzielone są one na:

**Zmienne wspólne dla VirtualBox i Proxmox VE:**

- `version` - wersja systemu Debian do instalacji (domyślnie 12.5.0)
- `iso` - adres URL obrazu ISO systemu Debian
- `checksum` - suma kontrolna SHA256 obrazu ISO
- `hostname` - nazwa hosta systemu (domyślnie 'eksporter-kart')
- `domain` - domena systemu (domyślnie 'local')
- `disable_dhcp` - ustaw na `false` jeśli chcesz użyć DHCP
- `ipaddress` - adres IP w statycznej konfiguracji sieci
- `netmask` - maska sieci
- `gateway` - brama sieci
- `nameservers` - serwery nazw

**Zmienne specyficzne dla VirtualBox:**

- `vm_name` - nazwa maszyny wirtualnej (domyślnie 'eksporter-kart')
- `cpus` - liczba CPU (domyślnie 2)
- `memory`- ilość pamięci RAM w MB (domyślnie 4096)
- `disk_size` - rozmiar dysku w MB (domyślnie 50000)

**Zmienne specyficzne dla Proxmox VE:**

- `proxmox_disk_size` - rozmiar dysku w GB (domyślnie 50)
- `proxmox_url` - URL Proxmox API (https://<adres_ip_maszyny>:8006/api2/json `dostosuj do własnych ustawień`)
- `proxmox_user` - nazwa użytkownika Proxmox (`dostosuj`)
- `proxmox_password` - hasło użytkownika Proxmox (`dostosuj`)
- `proxmox_node` - nazwa węzła Proxmox (`dostosuj`)
- `proxmox_vm_id` - ID maszyny wirtualnej/szablonu  (domyślnie 8003 - `dostosuj jeśli występuje konflikt VM ID`)
- `proxmox_cpu_type` - typ CPU (domyślnie 'host')
- `proxmox_network_bridge` - nazwa mostka sieciowego, np. vmbr0, vmbr1 (`dostosuj`)

**Zmienne specyficzne dla aplikacji Eksporter KART:**

- `SMB_HOST` - adres IP lub nazwa hosta z udziałem SMB (`dostosuj`)
- `SMB_PATH` - ścieżka do udziału SMB (`dostosuj`)  
- `SMB_USERNAME` - nazwa użytkownika SMB (`dostosuj`)
- `SMB_PASSWORD` - hasło użytkownika SMB (`dostosuj`)
- `SMB_WORKGROUP` - nazwa grupy roboczej SMB (`dostosuj`)
- `SQLSRV_HOST` - adres IP lub nazwa hosta z MS SQL Server (`dostosuj`)
- `SQLSRV_USERNAME` - nazwa użytkownika MS SQL (`dostosuj`)
- `SQLSRV_PASSWORD`- hasło użytkownika MS SQL (`dostosuj`)
- `SQLSRV_DATABASE`- nazwa bazy danych MS SQL (`dostosuj`)

### Skrypt eksporter-kart_setup.sh

Podczas budowania maszyny wirtualnej, Packer wykonuje polecenia zawarte w pliku szablonu `eksporter-kart.pkr.hcl` oraz skrypt `http/eksporter-kart_setup.sh`. Główne czynności, które przeprowadza to:

1. Aktualizacja systemu i instalacja wymaganych pakietów za pomocą apt.
2. Instalacja sterownika Microsoft ODBC Driver 18 i narzędzia wiersza poleceń mssql-tools18.
3. Kompilacja i instalacja modułów PHP `sqlsrv` i `pdo_sqlsrv`.
4. Kompilacja i instalacja modułu PHP `smbclient`.
5. Tworzenie bazy danych MySQL 'eksporter-kart' i użytkownika 'eksporter-kart' z hasłem.
6. Klonowanie repozytorium aplikacji Eksporter KART z prywatnego repozytorium GitHub.
7. Instalacja zależności i budowa front-endu aplikacji Eksporter KART.

### Skrypt cleanup.sh

Na koniec Packer wykonuje skrypt `http/cleanup.sh`, który czyści system po instalacji.

### Uruchamianie Packera

Po wykonaniu czynności wstępnych, zainstalowaniu Packera, dostosowaniu ustawień sieci w preseed.cfg i zmiennych w pliku `eksporter-kart.pkr.hcl`, możemy zacząć właściwy proces instalacji. Możliwe są 2 opcje:
  
1. Utworzenie obrazu maszyny dla VirtualBox (konieczna instalacja VirtualBox na maszynie z Packerem)

   ```sh
   packer build -only debian.virtualbox-iso.debian eksporter-kart.pkr.hcl
   ```

   Packer utworzy virtualną maszynę Debian i skonfiguruje ją zgodnie ze zdefiniowanym planem. Na koniec wyeksportuje obraz OVA maszyny w katalogu `output-debian/`.
   Obraz można zaimportować do VirtualBox.

2. Utworzenie szablonu VM na Proxmox VE

   ```sh
   packer build -only debian.proxmox-iso.debian eksporter-kart.pkr.hcl
   ```

   Packer utworzy maszynę wirtualną Debiana na podanym węźle Proxmox, skonfiguruje system i aplikację zgodnie z planem, a na koniec utworzy z niej szablon do klonowania.

   Aby sklonować utworzony szablon na Proxmox VE, można użyć interfejsu (pełen klon) lub polecenia `qm clone`, np.:

   ```sh
   qm clone 8003 200 --name eksporter-kart-vm --full --storage local-lvm
   ```

   To sklonuje szablon o ID 8003 do maszyny o ID 200, nazwie "eksporter-kart-vm". Klon będzie pełny (nie linkowany), a dyski zostaną utworzone na storage "local-lvm".

   Następnie uruchom sklonowaną maszynę:

   ```sh
   qm start 200
   ```

Po udanym procesie możemy uruchomić gotową maszynę z aplikacją Eksporter KART. Aplikacja dostępna będzie pod adresem: `http://<adres_ip_maszyny>`.
Domyślny login i hasło do aplikacji to `admin`.

## Instrukcja ręcznej instalacji aplikacji Eksporter KART na Linuxie

Poniższa instrukcja przedstawia kroki niezbędne do ręcznej instalacji aplikacji Eksporter KART na systemie Linux.

### Wymagania systemowe

Przed przystąpieniem do instalacji aplikacji Eksporter KART, upewnij się, że Twój system spełnia następujące wymagania:

- System operacyjny: Linux (W tym przykładzie Debian 12)
- PHP: wersja >= 8.2
- Composer
- Yarn
- Serwer bazy danych: MariaDB w wersji 10.3 lub nowszej lub MySQL
- Serwer web: Nginx lub Apache 2 (zalecam Nginx)
- Supervisor

Upewnij się, że oprócz powyższych wymagań, masz zainstalowane następujące pakiety i biblioteki:
(Dla wersji PHP8.3 i Nginx)

- linux-headers-$(uname -r)
- unixodbc-dev
- build-essential
- dkms
- git
- php8.3-fpm lub libapache2-mod-php w zależności od użytego serwera http.
- php8.3-cli
- php8.3-curl
- php-composer-pcre
- php8.3-dom
- php8.3-common
- php8.3-amqp
- php8.3-uuid
- php8.3-readline
- php8.3-raphf
- php8.3-odbc
- php8.3-zip
- php8.3-msgpack
- php8.3-http
- php8.3-opcache
- php8.3-mysql
- php8.3-dev
- cifs-utils
- libsmbclient-dev
- php8.3-xml
- php8.3-intl
- libpcre2-posix3

Jeśli któryś z powyższych pakietów nie jest zainstalowany, zostanie on automatycznie zainstalowany podczas wykonywania instrukcji instalacji.

### 1. Aktualizacja systemu i instalacja pakietów

   ```sh
   sudo apt update && sudo apt upgrade -y
   sudo apt install -y composer linux-headers-$(uname -r) unixodbc-dev build-essential dkms git mariadb-server nginx supervisor php8.3-fpm php8.3-cli php8.3-gnupg php8.3-curl php-composer-pcre php8.3-dom php8.3-common php8.3-amqp php8.3-uuid php8.3-readline php8.3-raphf php8.3-odbc php8.3-zip php8.3-msgpack php8.3-http php8.3-opcache php8.3-mysql cifs-utils libsmbclient-dev php8.3-dev php8.3-xml php8.3-intl libpcre2-posix3 apt-transport-https
   sudo ln -s /usr/bin/yarnpkg /usr/bin/yarn
   ```

### 2. Dodawanie wymaganych repozytoriów apt

Dodaje klucz GPG i repozytorium apt dla Microsoft SQL Server i repozytorium apt dla PHP:

   ```sh
   curl https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc
   sudo echo "deb [arch=amd64,arm64,armhf] https://packages.microsoft.com/debian/12/prod bookworm main" | sudo tee /etc/apt/sources.list.d/mssql-release.list  
   sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
   sudo echo "deb https://packages.sury.org/php/ bookworm main" | sudo tee /etc/apt/sources.list.d/php.list
   sudo apt update
   ```

### 3. Instalacja sqlsrv driver

Instalacja oprogramowanie Microsoft, niezbędnego do pracy z MS SQL Server:

   ```sh
   sudo -E ACCEPT_EULA=Y apt install -y msodbcsql18
   sudo -E ACCEPT_EULA=Y apt install -y mssql-tools18
   echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' | sudo -E tee -a /etc/profile
   sudo -E pecl channel-update pecl.php.net
   sudo -E pecl install sqlsrv
   sudo -E pecl install pdo_sqlsrv
   printf "; priority=20\nextension=sqlsrv.so\n" | sudo tee /etc/php/8.3/mods-available/sqlsrv.ini
   printf "; priority=30\nextension=pdo_sqlsrv.so\n" | sudo tee /etc/php/8.3/mods-available/pdo_sqlsrv.ini
   sudo -E phpenmod -v 8.3 sqlsrv pdo_sqlsrv
   ```

### 4. Instalacja rozszerzenia smbclient

   ```sh
   sudo pecl install smbclient
   printf "; priority=20\nextension=smbclient.so\n" | sudo tee /etc/php/8.3/mods-available/smbclient.ini
   sudo phpenmod -v 8.3 smbclient
   sudo systemctl reload php8.3-fpm
   ```

### 5. Tworzenie bazy danych i użytkownika

  Tworzy bazę MariaDB `eksporter-kart` z użytkownikiem `eksporter-kart`, autoryzowanym hasłem `0K3Exp0ne`.

   ```sh
   sudo mysql -u root <<EOF
   CREATE DATABASE \`eksporter-kart\`;
   CREATE USER 'eksporter-kart'@'localhost' IDENTIFIED BY '0K3Exp0ne';
   GRANT ALL PRIVILEGES ON \`eksporter-kart\`.* TO 'eksporter-kart'@'localhost';
   FLUSH PRIVILEGES;
   EOF
   ```

### 6. Instalacja aplikacji z prywatnego repozytorium GitHub

   ```sh
   sudo -E chmod 0777 /var/www
   cd /var/www
   git clone https://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:x-oauth-basic@github.com/xebeeche/eksporter-kart
   cd eksporter-kart
   cp ./.env.example ./.env
   composer install
   php artisan migrate:fresh --seed --force
   php artisan key:generate
   php artisan storage:link
   yarn install
   yarn build
   php artisan optimize
   ```

   Dopisanie konfiguracji do pliku `.env`

   ```sh
   echo "Dodaje zmienne do pliku .env"
   echo "SMB_HOST=<dostosuj>" >> /var/www/eksporter-kart/.env
   echo "SMB_PATH=<dostosuj>" >> /var/www/eksporter-kart/.env
   echo "SMB_USERNAME=<dostosuj>" >> /var/www/eksporter-kart/.env
   echo "SMB_PASSWORD=<dostosuj>" >> /var/www/eksporter-kart/.env
   echo "SMB_WORKGROUP=<dostosuj>" >> /var/www/eksporter-kart/.env
   echo " " >> /var/www/eksporter-kart/.env
   echo "SQLSRV_HOST=<dostosuj>" >> /var/www/eksporter-kart/.env
   echo "SQLSRV_USERNAME=<dostosuj>" >> /var/www/eksporter-kart/.env
   echo "SQLSRV_PASSWORD=<dostosuj>" >> /var/www/eksporter-kart/.env
   echo "SQLSRV_DATABASE=<dostosuj>" >> /var/www/eksporter-kart/.env
   sudo chown -R www-data:www-data /var/www/eksporter-kart
   ```

### 7. Przykładowa konfiguracja Nginx

`/etc/nginx/sites-available/eksporter-kart`

   ```sh
   server {
    listen 0.0.0.0:80;
    server_name _;
    root /var/www/eksporter-kart/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.php index.html index.htm;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock; # Dostosuj do swojej wersji PHP
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
   }
   ```

### 8. Konfiguracja Supervisor

   Supervisor jest serwerem uruchamiającym workera frameworku Laravel, odpowiedzialnego za wykonywanie zadań:

   1. Importu kart do repozytoriów.

   2. Kopiowania plików kart odpowiedzi na serwer linii ABBYY.

   3. Wykonanie eksportu kart odpowiedzi do systemu ONE.

   `/etc/supervisor/conf.d/eksporter-kart-worker.conf`

   ```sh
   [program:eksporter-kart-worker]
   process_name=%(program_name)s_%(process_num)02d
   command=php /var/www/eksporter-kart/artisan queue:work --daemon --timeout=600
   autostart=true
   autorestart=true
   user=www-data
   numprocs=1
   redirect_stderr=true
   stdout_logfile=/var/log/supervisor/eksporter-kart-worker.log
   stderr_logfile=/var/log/supervisor/eksporter-kart-worker.err.log
   stdout_logfile_maxbytes=50MB
   stdout_logfile_backups=10
   stderr_logfile_maxbytes=50MB
   stderr_logfile_backups=10
   ```

Domyślny login i hasło do aplikacji to `admin`. Hasło należy zmienić po pierwszym logowaniu [Opis działania aplikacji Eksporter KART](OPIS.md).

## Czynności poinstalacyjne

### Zmiana haseł użytkownika `root` i `packer`

Ważnym krokiem w zabezpieczeniu systemu jest zmiana domyślnych haseł użytkowników `root` oraz `packer`. Domyślne hasła są często celem ataków, dlatego ich zmiana na silne, unikalne hasła jest kluczowym elementem zabezpieczenia systemu.

Domyślne hasła:

`root`: r00tpass

`packer`: packer - użytkownik `packer` ma możliwość wykonywania polecenia `sudo` bez podawania hasła `root`

Przykładowe polecenie szybkiej zmiany hasła:

```sh
echo "root:NoweSilneHasloRoot" | sudo chpasswd
echo "packer:NoweSilneHasloPacker" | sudo chpasswd
```

### Dostęp do systemu i generowanie kluczy SSH

Instalacja konfiguruje prosty firewall `ufw`, który blokuje wszystkie połączenia przychodzące, oprócz portów `22` i `80`. Za pomocą `SSH` można się dostać do systemu jako użytkownik `packer` (logowanie po `SSH` dla `root` jest niemożliwe). Zaleca się wygenerowanie kluczy dostępu SSH, aby dodatkowo zabezpieczyć dostęp do systemu. Poniżej znajdują się kroki, jak to zrobić:

1. **Generowanie klucza SSH na lokalnej maszynie**:
   Na lokalnej maszynie, z której chcesz się logować do systemu, wygeneruj parę kluczy SSH. Użyj następującego polecenia:

   ```sh
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

   Podczas generowania klucza, zostaniesz poproszony o podanie nazwy pliku, w którym ma być zapisany klucz (domyślnie `~/.ssh/id_rsa`). Możesz również ustawić hasło zabezpieczające klucz prywatny.

2. **Skopiowanie klucza publicznego na zdalny serwer**:
   Skopiuj wygenerowany klucz publiczny na zdalny serwer, aby umożliwić bezpieczne logowanie się. Użyj następującego polecenia:

   ```sh
   ssh-copy-id packer@<IP_adres_serwera>
   ```

   Zastąp `<IP_adres_serwera>` adresem IP zdalnego serwera.

3. **Konfiguracja autoryzowanych kluczy na zdalnym serwerze**:
   Alternatywnie, jeśli `ssh-copy-id` nie jest dostępne, możesz ręcznie skopiować zawartość pliku `~/.ssh/id_rsa.pub` z lokalnej maszyny do pliku `~/.ssh/authorized_keys` na zdalnym serwerze.

   ```sh
   cat ~/.ssh/id_rsa.pub | ssh packer@<IP_adres_serwera> 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
   ```

4. **Zmiana uprawnień na zdalnym serwerze**:
   Upewnij się, że katalog `.ssh` oraz plik `authorized_keys` mają odpowiednie uprawnienia:

   ```sh
   ssh packer@<IP_adres_serwera>
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

5. **Testowanie logowania za pomocą klucza SSH**:
   Po poprawnym skopiowaniu klucza publicznego i ustawieniu uprawnień, możesz zalogować się na zdalny serwer bez użycia hasła:

   ```sh
   ssh packer@<IP_adres_serwera>
   ```

6. **Wyłączenie logowania za pomocą hasła** (opcjonalne, ale zalecane):
   Aby dodatkowo zabezpieczyć system, wyłącz logowanie za pomocą hasła. Edytuj plik konfiguracyjny SSH na zdalnym serwerze `/etc/ssh/sshd_config` i zmień lub dodaj następujące linie:

   ```plaintext
   PasswordAuthentication no
   PermitRootLogin no
   ```

   Następnie zrestartuj usługę SSH, aby zastosować zmiany:

   ```sh
   sudo systemctl restart sshd
   ```

## Podsumowanie

Zmiana domyślnych haseł oraz konfiguracja dostępu za pomocą kluczy `SSH` znacząco zwiększa bezpieczeństwo systemu. Upewnij się, że klucze `SSH` są przechowywane w bezpiecznym miejscu i że masz kopie zapasowe kluczy prywatnych. Wyłączenie logowania za pomocą hasła minimalizuje ryzyko nieautoryzowanego dostępu do systemu.

## [Licencja](LICENCE.md)

OPROGRAMOWANIE TO JEST DOSTARCZANE PRZEZ CENTRALNĄ KOMISJĘ EGZAMINACYJNĄ, WŁAŚCICIELI PRAW AUTORSKICH ORAZ WSPÓŁTWÓRCÓW W STANIE "TAK JAK JEST". WSZELKIE WYRAŹNE LUB DOROZUMIANE GWARANCJE, W TYM TE, NIE OGRANICZAJĄCE SIĘ DO DOROZUMIANYCH GWARANCJI WARTOŚCI HANDLOWEJ I PRZYDATNOŚCI DO OKREŚLONEGO CELU, SĄ WYŁĄCZONE. W ŻADNYM WYPADKU CENTRALNA KOMISJA EGZAMINACYJNA, WŁAŚCICIELE PRAW AUTORSKICH ANI WSPÓŁTWÓRCY NIE PONOSZĄ ODPOWIEDZIALNOŚCI ZA JAKIEKOLWIEK BEZPOŚREDNIE, POŚREDNIE, PRZYPADKOWE, SPECJALNE, PRZYKŁADOWE LUB WYNIKOWE SZKODY (W TYM, ZAKUP ZASTĘPCZYCH TOWARÓW LUB USŁUG; UTRATA MOŻLIWOŚCI UŻYTKOWANIA, DANYCH LUB ZYSKÓW; PRZERWY W DZIAŁALNOŚCI), WYNIKAJĄCE W JAKIKOLWIEK SPOSÓB Z UŻYCIA TEGO OPROGRAMOWANIA, BEZ WZGLĘDU NA PRZYCZYNĘ I NA JAKIEJKOLWIEK TEORIE ODPOWIEDZIALNOŚCI, CZY TO UMOWNE, CZY Z DELIKTU (W TYM ZANIEDBANIA LUB INNEGO), NAWET JEŚLI NIE ZOSTALI POINFORMOWANI O MOŻLIWOŚCI WYSTĄPIENIA TAKICH SZKÓD.

(c) 2024 CENTRALNA KOMISJA EGZAMINACYJNA. Wszelkie prawa zastrzeżone.
