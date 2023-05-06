# magicMirror2

Используемое железо: RPI 3B+, SD на 8 Гб.

## Инструкция
Основная ссылка: [MagicMirror2](https://magicmirror.builders/)

API: [docs](https://docs.magicmirror.builders/)

### Подготовка
1. Установить систему при помощи PI Imager - установка обычной системы. Желательно сразу установить необходимые параметры - подключить SSH, подключиться к сети WIFI. 
2. Подключить прошитую SD флешку в RPI, подключить HDMI, подключить питание. Дождаться полной загрузки.
3. Далее подключаемся через SSH:
```
ssh pi@mirror.local
```
где mirror - имя RPI.
4. Обновляем систему:
```
sudo apt update
sudo apt full-upgrade
```
Если не работает (2023 год, не качает с архивов), то изменить код на следующий:
```
sudo apt -o Acquire::ForceIPv4=true update
sudo apt -o Acquire::ForceIPv4=true full-upgrade
```
Так же могут быть ошибки с подключением:
> E: Не удалось получить доступ к файлу блокировки /var/lib/dpkg/lock-frontend - open (11: Ресурс временно недоступен) 

Тогда прописывать последовательно:
```
sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock
sudo rm /var/lib/dpkg/lock-frontend
```

5. Перезагружаем RPI:
```
sudo reboot
```

6. Ориентация дисплея по дефолту - горизонтальная. Если нужна портретная, то 
```
sudo nano /usr/share/dispsetup.sh
```
изменить 
```
#!/bin/sh
xrandr --output HDMI-1 --rotate right
exit 0
```
перезагрузить
```
sudo reboot
```

7. Устанавливает MagicMirror:
```
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install -y nodejs
```

НЕ ЗАБЫВАЕМ, ЧТО ЕСЛИ НЕ КАЧАЕТ С АРХИВОВ - ДОБАВЛЯТЬ 
```
-o Acquire::ForceIPv4=true
```

8. Клонируем:
```
git clone https://github.com/MichMich/MagicMirror
cd MagicMirror/
npm run install-mm
```

9. Копируем образец файла к себе
```
cp config/config.js.sample config/config.js
```

10. Запускаем приложение:
```
npm run start
```

### АВТОСТАРТ MagicMirror при запуске RPI
```
sudo npm install -g pm2
pm2 startup
```
Далее прямо в консоли будет предложение скопировать строку и вставить ее - делаем:
```
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u pi --hp /home/pi
```
Создаем скрипт:
```
cd
sudo nano mm.sh
```
Вносим данные:
```
cd ./MagicMirror
DISPLAY=:0 npm start
```
Выдаем права:
```
sudo chmod +x mm.sh
```
PM2 теперь рулит процессом запуска.
Запускаем и сохраняем:
```
pm2 start mm.sh
pm2 save
```
RPI по умолчанию включает ждущий режим, если не было активности пользователя. Исправить это:
```
sudo nano /etc/lightdm/lightdm.conf
```
В секцию [Seat:*] (или [SeatDefaults]) добавляю:
```
xserver-command=X -s 0 dpms
```
Перезагружаю RPI.

### Работа с конфигурационным файлом
Основной файл `config.js` и редактировать его можно так:
```
sudo nano MagicMirror/config/config.js
```
