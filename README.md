## Пользователи и группы. Авторизация и аутентификация

https://otus.ru/learning/253707/

#### <a name='toc'>Содержание</a>

1. [Типы пользователей](#1)
2. [Управление пользователями](#2)
3. [PAM](#3)
4. Как работает PAM в Linux?
5. Как настроить PAM в Linux?
6. Лабораторная работа
7. [Дополнительные источники](#recommended_sources)

## [[⬆]](#toc) <a name='1'>Типы пользователей</a>

В Linux пользователей можно разделить на три типа:

- **root** - суперпользователь, который имеет доступ ко всем возможностям вашей системы, к выполнению любой операции в ней. Он нужен для того, чтобы выполнять все системные функции, будь то установка программ, изменение драйверов и настроек, запуск некоторых критических программ.
- **Системные пользователи** - системные процессы с учетной записью в системе.
- **Обычные пользователи** - физические, реальные пользователи.


## [[⬆]](#toc) <a name='2'>Управление пользователями</a>

**Для управления пользователями в Linux можно использовать следующие команды:**

1. **Создание пользователей.** Для этого применяется низкоуровневая утилита `useradd`. Например, чтобы создать пользователя с именем «username», необходимо ввести: `useradd username`.
2. **Установка пароля.** Для этого используется команда passwd: `passwd username`, где username — это имя пользователя, для которого устанавливается пароль.
3. **Просмотр списка пользователей.** Для этого можно воспользоваться командой cut: `cut -d: -f1 /etc/passwd`. Эта команда считывает содержимое файла **/etc/passwd**, который содержит информацию о всех пользователях.
4. **Удаление пользователя.** Для этого используется команда userdel: `userdel username`. Однако по умолчанию она не удаляет файлы с домашнего каталога пользователя. Чтобы удалить их, можно использовать опцию -r: `userdel -r username`.
5. **Изменение информации о пользователе.** Изменить информацию о пользователе можно при помощи команды usermod. Например, для изменения домашнего каталога пользователя потребуется воспользоваться командой: `usermod -d /новый/путь/к/каталогу username`.

Обычные пользователи не могут создавать, удалять или изменять информацию о пользователях и группах, но если у обычного пользователя присутствуют права sudo, то он также сможет управлять пользователями в системе. 


## [[⬆]](#toc) <a name='3'>PAM</a>

**PAM (Pluggable Authentication Modules) — набор компонентов, предоставляющих программный интерфейс для авторизации пользователей в Linux.** Используется везде, где требуется аутентификация пользователя или проверка его прав, например, при подключении через SSH или FTP, а также при повышении привилегий через команду sudo.

**Модули PAM** находятся в директории lib/security для старых операционных систем типа CentOS и в директории /usr/lib/x86_64-linux-gnu/security для современных ОС вроде последних релизов Ubuntu.

**Конфигурационные файлы PAM** для различных приложений находятся в папке /etc/pam.d.

**Некоторые типы модулей PAM:**

- **auth** — аутентификация пользователя. Проверяют, точно ли пользователь является тем, за кого себя выдаёт.
- **account** — проверка возможности доступа к серверу. Уточняют, может ли пользователь в данный момент получить доступ к сервису или службе.
- **password** — обновление механизма аутентификации. Отвечают за обновление пароля пользователя.
- **session** — действия при входе. Открываются и закрываются в рамках идентификации, блокируют действия пользователя и производят очистку после завершения его сессии.


## [[⬆]](#toc) <a name='4'>Как работает PAM в Linux?</a>
https://losst.pro/nastrojka-pam-v-linux

## [[⬆]](#toc) <a name='5'>Как настроить PAM в Linux</a>
https://losst.pro/nastrojka-pam-v-linux


## [[⬆]](#toc) <a name='6'>Лабораторная работа</a>

1. https://hashicorp-releases.yandexcloud.net/vagrant/
2. Создадим Vagrantfile
3. vagrant up
4. vagrant ssh
5. sudo -i
6. sudo useradd otusadm && sudo useradd otus
7. echo "Otus2022!" | sudo passwd --stdin otusadm && echo "Otus2022!" | sudo passwd --stdin otus
8. Создаём группу admin: sudo groupadd -f admin
9. usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin


После создания пользователей, нужно проверить, что они могут подключаться по SSH к нашей ВМ. Для этого пытаемся подключиться с хостовой машины: 
ssh otus@192.168.57.10

1. Проверим, что пользователи root, vagrant и otusadm есть в группе admin:
```
cat /etc/group | grep admin
```

3. Создадим файл-скрипт /usr/local/bin/login.sh
```
vim /usr/local/bin/login.sh
```
```
#!/bin/bash
#Первое условие: если день недели суббота или воскресенье
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
 #Второе условие: входит ли пользователь в группу admin
 if getent group admin | grep -qw "$PAM_USER"; then
        #Если пользователь входит в группу admin, то он может подключиться
        exit 0
      else
        #Иначе ошибка (не сможет подключиться)
        exit 1
    fi
  #Если день не выходной, то подключиться может любой пользователь
  else
    exit 0
fi
```
3. Добавим права на исполнение файла: 
```
chmod +x /usr/local/bin/login.sh
```

4. Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт:
```
echo "auth required pam_exec.so debug /usr/local/bin/login.sh" >> /etc/pam.d/sshd
```






#### 5. [[⬆]](#toc) <a name='recommended_sources'>Дополнительные источники</a>

1. [Управление пользователями](https://firstvds.ru/technology/linux-user-management)
2. [Мандатное управление доступом](https://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D0%BD%D0%B4%D0%B0%D1%82%D0%BD%D0%BE%D0%B5_%D1%83%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5_%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF%D0%BE%D0%BC)
3. [Пользователи](https://www.altlinux.org/%D0%A3%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5_%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D1%8F%D0%BC%D0%B8)
4. [Настройка PAM в Linux](https://losst.pro/nastrojka-pam-v-linux)
