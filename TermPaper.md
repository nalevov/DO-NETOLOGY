# Курсовая работа по итогам модуля "DevOps и системное администрирование"

# Задание:

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

# Задание: 

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

# Задание: 

3. Установите hashicorp vault ([инструкция по ссылке](https://learn.hashicorp.com/tutorials/vault/getting-started-install?in=vault/getting-started#install-vault)).

### Решение:

Устанавливаем по инструкции с сайта 

```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vault
```
![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%203.1.png)

Проверяем работоспособность 

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%203.2.png)

# Задание: 

4. Cоздайте центр сертификации по инструкции ([ссылка](https://learn.hashicorp.com/tutorials/vault/pki-engine?in=vault/secrets-management)) и выпустите сертификат для использования его в настройке веб-сервера nginx (срок жизни сертификата - месяц).

### Решение:

1. Создаем по инструкции
```
sudo apt -y install jq
vault server -dev -dev-root-token-id root
```
![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%204.1.png)

2. Открываем дополнительный терминал 

```
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=root
```
3. Создаем сертификаты 

>Создание Root CA
```
# Включение механизма секретов pki в пути pki_root
vault secrets enable -path=pki_root pki
# Установка времени жизни для сертификатов 
vault secrets tune -max-lease-ttl=87600h pki_root
# Создание корневого сертификата
vault write -field=certificate pki_root/root/generate/internal common_name="netology.devops" ttl=87600h > RootCA.crt
# Добавление URL адреса CA и точку распределения
vault write pki_root/config/urls issuing_certificates="$VAULT_ADDR/v1/pki/ca" crl_distribution_points="$VAULT_ADDR/v1/pki/crl"
```

>Добавление промежуточного СА
```
vault secrets enable -path=pki_intermediate pki
# Установка время жизни для сертификатов 
vault secrets tune -max-lease-ttl=43800h pki_intermediate
# Генерируем CSR
vault write -format=json pki_intermediate/intermediate/generate/internal common_name="netology.devops Intermediate CA" | jq -r '.data.csr' > IntermediateCA.csr
# Подпишем закрытым ключом RootCA промежуточный сертификат и сохраним
vault write -format=json pki_root/root/sign-intermediate csr=@IntermediateCA.csr format=pem_bundle ttl="43800h" | jq -r '.data.certificate' > IntermediateCA.pem
# После того, как CSR подписан и получен сертификат, последний можно испортировать обратно в Vault
vault write pki_intermediate/intermediate/set-signed certificate=@IntermediateCA.pem
```

>Роль и выпуск сертификата
```
# Добавить роль, которая разрешает поддомены netology.devops сроком жизни до 30 дней
vault write pki_intermediate/roles/netology-dot-devops allowed_domains="netology.devops" allow_subdomains=true max_ttl="720h"
# Создать сертификат на 30 дней для доменного имени vault.netology.devops
vault write -format=json pki_intermediate/issue/netology-dot-devops common_name="vault.netology.devops" alt_names="vault.netology.devops" > vault.netology.devops.crt
# Сохраняем сертификат в правильном формате
cat vault.netology.devops.crt | jq -r '.data.private_key' > private.pem
cat vault.netology.devops.crt | jq -r '.data.certificate' > cert.crt
cat vault.netology.devops.crt | jq -r '.data.ca_chain[]' >> cert.crt
```

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%204.2.png)


# Задание: 

5. Установите корневой сертификат созданного центра сертификации в доверенные в хостовой системе.

### Решение:

1. Скопируем сертификат в общую папку

```
cp RootCA.crt /vagrant/
```

2. В консоли Сертификаты импортируем сертификат из общей папки

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%205.png)

# Задание:

6. Установите nginx.

### Решение:

`sudo apt install nginx`

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%206.png)

# Задание:

7. По инструкции ([ссылка](https://nginx.org/en/docs/http/configuring_https_servers.html)) настройте nginx на https, используя ранее подготовленный сертификат:
  - можно использовать стандартную стартовую страницу nginx для демонстрации работы сервера;
  - можно использовать и другой html файл, сделанный вами;

### Решение:

1. Создаем директорию для хранения сертификатов

```
sudo mkdir /etc/nginx/ssl
sudo cp private.pem /etc/nginx/ssl/private.pem
sudo cp cert.crt /etc/nginx/ssl/cert.crt
```

2. Редактируем конфигурацию стандартной страницы

```
nano /etc/nginx/sites-available/default
server {
	listen				443 ssl;
	server_name         vault.netology.devops;
 	ssl_certificate     /etc/nginx/ssl/cert.crt;
	ssl_certificate_key /etc/nginx/ssl/private.pem;
	ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers         HIGH:!aNULL:!MD5;
```

3. Сохраняем и перезапускаем сервис
sudo systemctl restart nginx

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%207.png)

# Задание:

8. Откройте в браузере на хосте https адрес страницы, которую обслуживает сервер nginx.

### Решение:

1. В файл C:\Windows\System32\drivers\etc\hosts добавим запись 127.0.0.1 vault.netology.devops

2. Откроем в браузере на хосте https адрес страницы, которую обслуживает сервер nginx.

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%208.png)

# Задание:

9. Создайте скрипт, который будет генерировать новый сертификат в vault:
  - генерируем новый сертификат так, чтобы не переписывать конфиг nginx;
  - перезапускаем nginx для применения нового сертификата.

### Решение:

Создаем скрипт `sudo nano /home/vagrant/5.py` со следующим содержанием

```
#!/bin/env python3
import OpenSSL,sys,os,json
from datetime import datetime

CRT_PATH = '/etc/nginx/ssl/cert.crt'
KEY_PATH = '/etc/nginx/ssl/private.pem'

def get_cert_expiredate(path):
    with open(path, 'rb') as fp:
        cert = fp.read()

    x509 = (OpenSSL.crypto.load_certificate(OpenSSL.crypto.FILETYPE_PEM, cert)).get_notAfter()
    return datetime.strptime(x509.decode("utf-8"),"%Y%m%d%H%M%SZ")

def write_file(data,path):
    file = open(path,"w")
    file.write(data)
    file.close

def err(msg: str):
    write_file(msg,'/home/vagrant/log.log')
    sys.exit(1)

exp_dt = get_cert_expiredate(CRT_PATH)

result = (exp_dt - datetime.now()).total_seconds() // 3600

print(f'{result} hours left until expiration')

if (result < 1000):
    try:
        data_json = json.loads(os.popen("vault write -format=json pki_intermediate/issue/netology-dot-devops common_name='vault.netology.devops' alt_names='vault.netology.devops'").read())
        private_key = data_json['data']['private_key']
        public_key = data_json['data']['certificate'] + "\n" + data_json["data"]['ca_chain'][0]

        write_file(private_key,KEY_PATH)
        write_file(public_key,CRT_PATH)

        os.popen("systemctl restart nginx")
    except Exception as e:
        print(e)
        err(f"error {e}\n")
```

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%BD%D0%B8%D0%B5%209.png)


# Задание:

10. Поместите скрипт в crontab, чтобы сертификат обновлялся какого-то числа каждого месяца в удобное для вас время.

### Решение:

1. Помещаем скрипт в crontab `sudo crontab -e` 

Для наглядности выбрали, чтобы сприпт выполнялся каждые 2 минуты

```
VAULT_ADDR=http://127.0.0.1:8200
VAULT_TOKEN=root

*/2**** /usr/bin/python3 /home/vagrant/5.py

```
![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%2010.png)

2. В итоге получаем:

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%2010.1.png)

![img.png](https://github.com/nalevov/DO-NETOLOGY/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%2010.2.png)
