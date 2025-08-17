## Архитектура
<img width="618" height="693" alt="image" src="https://github.com/user-attachments/assets/9df7fcad-d9c6-4561-a3c0-4350498fbc7d" />

## Настройка файлового сервера и монтирование директории
1. Создаем пользователя smbuser на VM-01
   ```console
   sudo adduser smbuser
   ```
   К этой учетной записи будем настраивать доступ. Пароль, который мы задали при создании пользователя, в дальнейшем нужно будет указать для доступа по smb-протоколу
   <br><br>
2. Проверяем что у созданного пользователя smbuser есть домашняя директория
   ```console
   sudo ls -lab /home/smbuser
   ```
   Домашняя директория пользователя smbuser будет использоваться как общая директория двух виртуальных машин
   <br><br>
3. Устанавливаем пакет samba на VM-01
   ```console
   sudo apt update && sudo apt upgrade -y
   sudo apt install samba -y
   ```
   <br>
4. В конфигурационном файле smb.conf указываем домашнюю директорию пользователя smbuser и делаем её доступной только для чтения
   ```console
   sudo nano /etc/samba/smb.conf
   ```
   <img width="412" height="184" alt="image" src="https://github.com/user-attachments/assets/054decb0-66a0-453b-af23-02b4061c5adb" />
   
   В конфигурационном файле указали что домашняя директория доступна только для чтения (read only = yes), ограничили доступ до директории - только для smbuser (valid user = smbuser) и добавили способ аутентификации по логину/паролю Samba (security = user)
   <br><br>
5. Создаем пользовательский пароль для доступа по smb-протоколу для smbuser
   ```console
   sudo smbpasswd -a smbuser
   ```
   Для удобства пароль будет одинаковым с системным паролем
   <br><br>
6. Рестартим сервис после изменения конфигурации на VM-01
   ```console
   sudo systemctl restart smbd.service
   ```
   <br>
7. Устанавливаем пакет cifs-utils для работы с smb-протоколом на VM-02
   ```console
   sudo apt update && sudo apt upgrade -y
   sudo apt install cifs-utils -y
   ```
   <br>
8. Устанавливаем пакет autofs для автоматического монтирования на VM-02
   ```console
   sudo apt install autofs –y
   ```
   <br>
9. Создадим точку монтирования на VM-02
   ```console
   sudo mkdir -p /mnt/smbuser_home/share
   ```
   <br>
10. Добавляем конфигурацию в auto.master на VM-02
    ```console
    sudo nano /etc/auto.master
    ```
    <img width="618" height="62" alt="image" src="https://github.com/user-attachments/assets/303cebf9-7516-41b0-9541-e974dd6ba4ea" />
    
    В конфигурационном файле указываем корневую директорию для монтирования (/mnt/smbuser_home), файл с описанием правил монтирования (/etc/auto.smbuser) и время через которое ресурс будет отмонтирован
    <br><br>
11. Добавляем конфигурацию в файл с описанием правил монтирования auto.smbuser на VM-02
    ```console
    sudo nano /etc/auto.smbuser
    ```
    <img width="618" height="97" alt="image" src="https://github.com/user-attachments/assets/c90b8624-4a0c-4e4f-beb1-9935374cb786" />

    Запись в одну строку, на скриншоте для удобства есть перенос
    
    В конфигурационном файле указываем точку монтирования (share, в итоге общая директория будет расположена по пути /mnt/smbuser_home/share), тип файловой системы CIFS (-fstype=cifs), креды для аутентификации (будут расположены в файле credentials), права на файлы и директории для клиента (дублируем права с сервера read only для корректной работы) и путь к самой шаре
    <br><br>
12. Создадим файл credentials с логином и паролем и защитим его
    ```console
    sudo mkdir /etc/samba/
    sudo nano /etc/samba/credentials
    sudo chmod 600 /etc/samba/credentials
    ```
    <img width="272" height="117" alt="image" src="https://github.com/user-attachments/assets/32bf53b6-9edf-46ff-bbe8-5d535f15922a" />
    <br><br>
12. Рестартим сервис autofs после изменения конфигурации
    ```console
    sudo systemctl restart autofs
    ```
    <br>
13. Проверяем что директория смонтирована на VM-02
    ```console
    cd /mnt/smbuser_home/share
    ls -la
    ```
    <img width="619" height="89" alt="image" src="https://github.com/user-attachments/assets/01e48a3b-66a7-49b8-8586-fd3aa2f411ab" />
    <br><br>
14. Проверяем что изменения директории под smbuser на VM-01 видны на VM-02 и что работает read only для VM-02
    
    VM-01
    ```console
    su smbuser
    cd ~
    echo «Hi!» > test.txt
    cat test.txt
    exit
    ```
    
    VM-02
    ```console
    cd /mnt/smbuser_home/share
    cat test.txt
    touch noperm.txt
    nano test.txt
    ```
    <br>
15. Проверяем что в директорию нельзя зайти с другими кредами на VM-02
    ```console
    sudo nano /etc/samba/credentials
    ```
    
    <img width="318" height="134" alt="image" src="https://github.com/user-attachments/assets/02557032-2b14-40bd-b204-ee41082926aa" />
    
    ```console
    sudo systemctl restart autofs
    cd /mnt/smbuser_home/share
    ```
    
    <img width="619" height="67" alt="image" src="https://github.com/user-attachments/assets/d80e24b7-9904-47da-a967-f158549105e8" />
    <br><br>
## Настройка прокси-сервера
1. Настраиваем VM-03 также как VM-02 в 1 задании (7-12)

2. На VM-02 и VM-03 запускаем сервер для отображения шары
   ```console
   python3 -m http.server --directory /mnt/smbuser_home/share &
   ```
   Проверить что сервер поднялся можно перейдя на http://10.11.0.140:8000 (VM-02) и http://10.12.1.25:8000 (VM-03). Сервера запускаем в фоновом режиме
   
   <img width="456" height="261" alt="image" src="https://github.com/user-attachments/assets/1e2b6b57-e06a-48d4-ad37-9af6e1d603e5" />
   
   <br>
3. Устанавливаем пакет nginx на VM-01
   ```console
   sudo apt update && sudo apt upgrade -y
   sudo apt install nginx -y
   ```
   <br>
4. Создаем конфиг для балансировки нагрузки в conf.d на VM-01
