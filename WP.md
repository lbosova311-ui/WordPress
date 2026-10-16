## Мета: Встановлення WordPress на власний вебсервер (LAMP)

### Середовище: Windows (хост), Oracle VirtualBox 7.2.12, Ubuntu Server 26.04.1 LTS, стек LAMP (Apache 2.4.66, MySQL 8.4.11, PHP 8.5.4), WordPress latest.

Встановлюємо Oracle VirtualBox та Extension Pack (add-on) тієї ж версії. Створюємо віртуальну машину `wp-server`, підключаємо образ `ubuntu-26.04.1-live-server-amd64.iso`. В налаштуваннях мережі (NAT) додаємо переадресацію портів: порт хоста 8080 на порт гостя 80 (сайт) та 2222 на 22 (SSH). Проходимо інсталятор Ubuntu Server (розмітка entire disk + LVM, відмічаємо "Install OpenSSH server"). Після встановлення підключаємось з Windows по SSH та розширюємо кореневий розділ на все вільне місце:

​```bash
ssh -p 2222 lisa@127.0.0.1
sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
​```

Оновлюємо систему та встановлюємо вебсервер, базу даних, PHP і потрібні WordPress розширення. Вмикаємо модуль `rewrite` (потрібен WordPress для permalinks), перезапускаємо Apache та перевіряємо версії і що сервіси запущені:

​```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl virtualbox-guest-utils
sudo a2enmod rewrite && sudo systemctl restart apache2
php -v; apache2 -v; mysql --version; systemctl is-active apache2 mysql
​```
<img width="1072" height="262" alt="image" src="https://github.com/user-attachments/assets/7e670331-1c11-47ae-918b-f32763b00ab1" />


Відкриваємо `http://localhost:8080` — Apache віддає стандартну сторінку, тобто вебсервер і переадресація портів працюють:

<img width="1002" height="423" alt="image" src="https://github.com/user-attachments/assets/6d8a5db4-3e31-4d6d-8661-6d244b887a1d" />


Заходимо в MySQL (`sudo mysql`) та створюємо базу і окремого користувача тільки для WordPress (права лише на свою базу, підключення тільки з localhost — принцип найменших привілеїв):

​```sql
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
SHOW DATABASES;
​```

<img width="818" height="475" alt="image" src="https://github.com/user-attachments/assets/f89c3ba8-32d0-4c6d-b424-a82fd9546a5b" />


Завантажуємо останню версію WordPress, розпаковуємо і копіюємо в кореневу папку сайту. Файли віддаємо користувачу вебсервера `www-data`:

​```bash
cd /tmp && wget https://wordpress.org/latest.tar.gz && tar -xzf latest.tar.gz
sudo rm -f /var/www/html/index.html
sudo cp -a /tmp/wordpress/. /var/www/html/
sudo chown -R www-data:www-data /var/www/html
ls -l /var/www/html
​```

<img width="871" height="502" alt="image" src="https://github.com/user-attachments/assets/df380d36-60e0-4c9d-9392-a2eb0206c8fd" />


Відкриваємо `http://localhost:8080` і вводимо параметри підключення до бази (Database Name: wordpress, Username: wpuser, Host: localhost, Table Prefix: wp_):

<img width="868" height="587" alt="image" src="https://github.com/user-attachments/assets/e5d1b7c5-d5ba-42f7-bad8-c992252d2494" />


Задаємо назву сайту та обліковий запис адміністратора:

<img width="875" height="672" alt="image" src="https://github.com/user-attachments/assets/fe62615f-fb9b-4f05-a9e1-3230035739b4" />

<img width="695" height="272" alt="image" src="https://github.com/user-attachments/assets/b7d00be1-fc50-49a3-a460-9d2466fbde45" />

