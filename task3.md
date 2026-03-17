Создаём папку и файл:
sudo mkdir -p /opt/app
sudo touch /opt/app/log.txt
 
Создаём скрипт:
sudo nano /usr/local/bin/write-log.sh

Пишим в редакторе:
#!/bin/bash
LOGFILE="/opt/app/log.txt"
while true; do
    RANDOM_STRING=$(cat /dev/urandom | LC_ALL=C tr -dc 'a-zA-Z0-9' | fold -w 20 | head -n 1)
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $RANDOM_STRING" >> "$LOGFILE"
    sleep 17
done

сохраняем, выходим.

Делаем скрипт исполняемым:
sudo chmod +x /usr/local/bin/write-log.sh
 
Проверяем работу скрипта:

sudo /usr/local/bin/write-log.sh

Ждем 30 секунд, нажимаем Ctrl+C
Смотрим результат:

cat /opt/app/log.txt
 
Добавляем в автозагрузку (launchd)
Создаём plist-файл:

sudo nano /Library/LaunchDaemons/com.user.logwriter.plist

Вставляем:
xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.logwriter</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/write-log.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>

Сохраняем, выходим

Загружаем службу:
sudo launchctl load /Library/LaunchDaemons/com.user.logwriter.plist

Проверяем:
sudo launchctl list | grep logwriter
 
Смотрим логи в реальном времени:

tail -f /opt/app/log.txt

Настройка logrotate (ротация логов)
Устанавливаем logrotate (Homebrew):

brew install logrotate

Создаём конфиг:

sudo nano /usr/local/etc/logrotate.d/app-log

Вставляем:

/opt/app/log.txt {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 644 root wheel
}

Сохраняем, выходим

Проверяем конфиг:

sudo logrotate -d /usr/local/etc/logrotate.d/app-log

Добавляем в cron:

sudo crontab -e

Вставляем строку:

0 0 * * * /usr/local/sbin/logrotate /usr/local/etc/logrotate.d/app-log

Сохраняем, выходим
