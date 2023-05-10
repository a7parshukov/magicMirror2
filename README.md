# Интеллектуальное зеркало: magicMirror2
## Железо
* Raspberry PI 3B+
* SD на 8 Гб
* Блок питания Raspberry PI (любой другой = low voltage problem)

## Инструкция по установке
### Полезные ссылки
* Основная ссылка: [MagicMirror2](https://magicmirror.builders/)
* API: [docs](https://docs.magicmirror.builders/)

### Подготовка
1. Установить систему при помощи PI Imager - установка обычной системы. Желательно сразу установить необходимые параметры - подключить SSH, подключиться к сети WIFI. 

2. Подключить прошитую SD флешку в RPI, подключить HDMI, подключить питание. Дождаться полной загрузки.

3. Далее подключаемся через SSH (putty или VSCode SSH):
```console
ssh pi@mirror.local
```
где mirror - имя RPI.

4. Обновляем систему:
```console
sudo apt update
sudo apt full-upgrade
```
**Если не работает (2023 год, не качает с архивов), то изменить код на следующий:**
```console
sudo apt -o Acquire::ForceIPv4=true update
sudo apt -o Acquire::ForceIPv4=true full-upgrade
```

Так же могут быть ошибки с подключением:
> E: Не удалось получить доступ к файлу блокировки /var/lib/dpkg/lock-frontend - open (11: Ресурс временно недоступен) 

Тогда прописывать последовательно:
```console
sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock
sudo rm /var/lib/dpkg/lock-frontend
```

5. Перезагружаем RPI:
```console
sudo reboot
```

6. Ориентация дисплея по дефолту - горизонтальная. Если нужна портретная, то 
```console
sudo nano /usr/share/dispsetup.sh
```
изменить 
```console
#!/bin/sh
xrandr --output HDMI-1 --rotate right
exit 0
```
перезагрузить
```console
sudo reboot
```

7. Устанавливает MagicMirror:
```console
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install -y nodejs
```

Если не качает с архивов, то добавлять 
```console
-o Acquire::ForceIPv4=true
```

8. Клонируем проект:
```console
git clone https://github.com/MichMich/MagicMirror
cd MagicMirror/
npm run install-mm
```

9. Копируем образец файла к себе
```console
cp config/config.js.sample config/config.js
```

10. Запускаем приложение:
```console
npm run start
```

### Настроить автоастарт MagicMirror при запуске RPI
```console
sudo npm install -g pm2
pm2 startup
```
Далее прямо в консоли будет предложение скопировать строку и вставить ее - делаем:
```console
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u pi --hp /home/pi
```
Создаем скрипт:
```console
cd
sudo nano mm.sh
```
Вносим данные:
```console
cd ./MagicMirror
DISPLAY=:0 npm start
```
Выдаем права:
```console
sudo chmod +x mm.sh
```
PM2 теперь рулит процессом запуска.
Запускаем и сохраняем:
```console
pm2 start mm.sh
pm2 save
```
RPI по умолчанию включает ждущий режим, если не было активности пользователя. Исправить это:
```console
sudo nano /etc/lightdm/lightdm.conf
```
В секцию `[Seat:*]` (или `[SeatDefaults]`) добавляю:
```console
xserver-command=X -s 0 dpms
```
Перезагружаем RPI.

## Работа с конфигурационным файлом
Основной файл `config.js` и редактировать его можно так:
```console
sudo nano MagicMirror/config/config.js
```