# Деплой приложения на django
Для работы приложений используется nginx, python3 и uwsgi, установленный через pip

## Первичная настройка сервера

- __Установка необходимого софта__

```bash
apt-get update
apt-get install python3-pip git nginx
pip3 install virtualenv uwsgi
```
- __Настройка uwsgi__

Для каждого приложения необходим ini файл, с помощью которого uwsgi будет запускать приложение.
Создадим папку для хранения этих файлов.

```bash
mkdir -p /etc/uwsgi/vassals
```

Запустим uwsgi в режиме демона, чтобы он автоматически подхватывал новые конфиги

```bash
/usr/local/bin/uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data --daemonize /var/log/uwsgi-emperor.log --pidfile /tmp/uwsgi.pid
```

__--pidfile /tmp/uwsgi.pid__ - путь к файлу, в котором хранится id процесса uwsgi, с его помощью можно управлять uwsgi с консоли.
Например, остановить процесс (остановятся все приложения):
```bash
/usr/local/bin/uwsgi --stop /tmp/uwsgi.pid
```

## Установка самого приложения

- __клонируем скелет проекта__

_Внимание. Не забываем поменять __project__ на свое название папки, так же надо будет поменять это название в последующих командах_

```bash
cd /var/www
git clone https://github.com/z-z/django-template.git project
```

- __устанавливаем зависимости проекта__

Зависимости каждого приложения будут в отдельном виртуальном окружении, чтоб не мешать друг другу. Виртуальное окружение надо создать через virtualenv, активировать его, установить зависимости, и деактивировать.

```bash
cd project
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
deactivate
```

Если надо установить еще зависимости, точно так же активируем окружение, устанавливаем зависимости, и деактивируем.

- __назначаем владельцем папки проекта пользователя www-data__

```bash
cd /var/www
chown -R www-data:www-data project
cd project
```

- Далее надо скопировать uwsgi.ini в /etc/uwsgi/vassals (или сделать символьную ссылку) и подредактировать там пути. Так же надо настроить nginx. Пример конфига в файле nginx.conf

- Перезапускаем nginx и готово.

__При такой настройке после перезагрузки сервера приложения будут слетать и их надо запускать вручную. Чтобы так не надо было делать, в файл /etc/rc.local, перед строкой “exit 0” добавляем:__

```bash
/usr/local/bin/uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data --daemonize /var/log/uwsgi-emperor.log --pidfile /tmp/uwsgi.pid
```
