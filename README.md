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
