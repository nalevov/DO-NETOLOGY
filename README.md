# Курсовая работа по итогам модуля "DevOps и системное администрирование"

## Задание

1. Создайте виртуальную машину Linux.

### Решение:
1 > cd g: 			- переходим на диск в котором будет развернута виртуальная машина
2 mkdir vagrant 		- создаем директорию, в которой будут храниться конфигурационные файлы Vagrant
3 cd vagrant/			- переходим в директорию с конфигурационными файлами
4 vagrant init		- выполняем vagrant init
5 nano Vagrantfile	или	- открываем файл конфигурации Vagrantfile в папке, созданной на этапе 4.2 с помощью любого текстового редактора в ОС WiNDOWS:
 				В строке  config.vm.box = "base" заменяем base на bento/ubuntu-20.04, сохраняем изменения
6 vagrant up			- запускаем ВМ. Ждем когда скачается образ и запустится ВМ 

В моем случае шаги 1-5 были выполнены в ходе предыдущей работы, запускам ВМ при помощи команды `vagrant up`, подключаемся `vagrant ssh` 

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%201.png)

## Задание 

2. Установите ufw и разрешите к этой машине сессии на порты 22 и 443, при этом трафик на интерфейсе localhost (lo) должен ходить свободно на все порты.

### Решение:
1. Добавляем нового пользователя `sudo adduser levov`

```
Adding user `levov' ...
Adding new group `levov' (1001) ...
Adding new user `levov' (1001) with group `levov' ...
Creating home directory `/home/levov' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for levov
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
```

2. Предоставляем права sudo новому пользователю `sudo usermod -aG sudo levov`
3. UFW устанавливается в Ubuntu по умолчанию. Если вы удалили UFW, вы можете установить его с помощью команды `sudo apt install ufw`.
4. Задаем правила по умолчанию - запрещаем все входящие соединения `sudo ufw default deny incoming` и разрешаем все исходящие `sudo ufw default allow outgoing`

```
levov@vagrant:/home/vagrant$ sudo ufw default deny incoming
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)
levov@vagrant:/home/vagrant$ sudo ufw default allow outgoing
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)
levov@vagrant:/home/vagrant$
```

5. Разрешаем сессии на порты 22 и 443 `sudo ufw allow `

```
levov@vagrant:/home/vagrant$ sudo ufw allow 22
Rules updated
Rules updated (v6)
levov@vagrant:/home/vagrant$ sudo ufw allow 443
Rules updated
Rules updated (v6)
```

6. Разрешаем трафик на интерфейсе localhost (lo)  `sudo ufw allow in on lo`

7. Включаем uwf `sudo ufw enable`

```
levov@vagrant:/home/vagrant$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

8. Проверяем статус `sudo ufw status verbose`

```
levov@vagrant:/home/vagrant$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22                         ALLOW IN    Anywhere
443                        ALLOW IN    Anywhere
Anywhere on lo             ALLOW IN    Anywhere
22 (v6)                    ALLOW IN    Anywhere (v6)
443 (v6)                   ALLOW IN    Anywhere (v6)
Anywhere (v6) on lo        ALLOW IN    Anywhere (v6)
```
![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%202.png)

## Задание 

3. Установите hashicorp vault ([инструкция по ссылке](https://learn.hashicorp.com/tutorials/vault/getting-started-install?in=vault/getting-started#install-vault)).

Устанавливаем по инструкции с сайта 

```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vault
```
![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%203.1.png)

Проверяем работоспособность 

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%203.2.png)


4. Cоздайте центр сертификации по инструкции ([ссылка](https://learn.hashicorp.com/tutorials/vault/pki-engine?in=vault/secrets-management)) и выпустите сертификат для использования его в настройке веб-сервера nginx (срок жизни сертификата - месяц).
5. Установите корневой сертификат созданного центра сертификации в доверенные в хостовой системе.
6. Установите nginx.
7. По инструкции ([ссылка](https://nginx.org/en/docs/http/configuring_https_servers.html)) настройте nginx на https, используя ранее подготовленный сертификат:
  - можно использовать стандартную стартовую страницу nginx для демонстрации работы сервера;
  - можно использовать и другой html файл, сделанный вами;
8. Откройте в браузере на хосте https адрес страницы, которую обслуживает сервер nginx.
9. Создайте скрипт, который будет генерировать новый сертификат в vault:
  - генерируем новый сертификат так, чтобы не переписывать конфиг nginx;
  - перезапускаем nginx для применения нового сертификата.
10. Поместите скрипт в crontab, чтобы сертификат обновлялся какого-то числа каждого месяца в удобное для вас время.

## Результат

Результатом курсовой работы должны быть снимки экрана или текст:

- Процесс установки и настройки ufw
- Процесс установки и выпуска сертификата с помощью hashicorp vault
- Процесс установки и настройки сервера nginx
- Страница сервера nginx в браузере хоста не содержит предупреждений 
- Скрипт генерации нового сертификата работает (сертификат сервера ngnix должен быть "зеленым")
- Crontab работает (выберите число и время так, чтобы показать что crontab запускается и делает что надо)
