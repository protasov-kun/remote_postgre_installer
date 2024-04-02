# Remote PostgreSQL Installer

Предлагаю вашему вниманию решение, кторое позволит осуществить установку и начальную настроку сервера ***PostgreSQL*** на базе операционной системы  ***GNU Linux***.

Данное решение представляет из себя скрипт ***Python***, который принимает необходимые параметры и обращается к системе управления конфигурациями ***Ansible***, для удаленной инсталяции и настройки сервера.

Для реализации этого решения нам понадобится операционная хост система на базе ядра ***Linux***. Предположим, что это ***Ubuntu***.

## Начнем

Поскольку для установки для установки ПО требуются права привелегированного пользователя, на удаленном хосте требуется предоставить полномичия пользователю, от имени которого будуд осуществлятся действия. 

Чтобы избедать ошибок доступа пользователь должен иметь права на выполнение команд суперпользователя на удаленном хосте через ***sudo***. Обычно для этого пользователь добавляется в группу ***sudoers*** и настроены необходимые правила доступа через ***sudo***.

Настройка ***sudo*** для пользователя включает в себя обычно добавление его в файл `/etc/sudoers` (через команду `visudo` для безопасности) или в файл в директории `/etc/sudoers.d/`. В этом файле определяются правила, позволяющие пользователю выполнять определенные команды от имени суперпользователя без ввода пароля или с вводом пароля (в зависимости от настроек).

### Теперь действуем на хост системе

Чтобы избежать парольной аутентификации создадим пару ключей ssh и передадим публичную часть ключа на удаленный хост:
```
ssh-keygen -t ed25519
ssh-copy-id -i ~/.ssh/id_ed25519.pub <логин_удвленного_пользователя>@<ip_адрес_или_hostname_удаленной_машины>
```

Установите необходимое ПО:
```
sudo apt-get update && sudo apt-get install ansible
```
вместе с пакетом ***Ansible*** на системе зависимостью установится интерпритатор ***Python3***.

В домашнем каталоге пользователя `/home/<USER>/` (или в какой вам удобно) добавьте локальную копию репозитория и перейдите в новый каталог:
```
git clone https://github.com/protasov-kun/remote_postgre_installer.git && cd remote_postgre_installer
```

### Все готово
В каталоге `/remote_postgre_installer` расположен файл скрипта `install_postgre.py,` который мы непосредственно будем вызывать и вспомогательный файлы для ***Ansible***, а так же файл с ***shell*** командой для тестирования работы сервера.

Вызов команды осуществляется из рабочего каталога:
```
python3 install_postgre.py <IP_адрес_или_hostname_удаленного_хоста>
```
после ввода команды в stdout появится сообщение `Enter the username for Ansible:` и вам потребуется указать пользователя, которому вы передали публичную часть ssh ключа.

Скрипт обновит данные в файлах `hosts` и `test.sh,` указав введенные вами значения, запустит выполение ***Ansible-playbook***, кторый в свою очередь использует ***Ansible-role***, расположенную в [публичном репозитории](https://github.com/protasov-kun/ansible-role-postgre-install) посредством ***Ansible-galaxy***, настроенном в файле `requirements.yml.`

В результате работы скрипта в ***stdout*** появится сообщение `PostgreSQL server installed and ready for remote job,` которое говорит само за себя.

Если в ходе выполнения программы возникнут ошибки, то в ***stdout*** так же появиться вывод, указывающий на каком этапе произошел сбой.