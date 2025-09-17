
# 🔒 Полное руководство по защите от утечек DNS и WebRTC

# Как защититься?
Выбирайте DNS себе
DNS servers to Google's 8.8.8.8/8.8.4.4 
либо
DNS servers to Cloudflare's 1.1.1.1/1.0.0.1

до (использовался ваш же айпишник)
<img width="386" height="22" alt="image" src="https://github.com/user-attachments/assets/c032a251-89bd-434a-b944-df23133e7125" />

заполняем как на фото 
Settings -> Network & internet -> Ethernet or WI-FI -> DNS server assignment (EDIT)

<img width="472" height="551" alt="image" src="https://github.com/user-attachments/assets/e60699a0-8db8-4d1f-9d2c-751cc7440faa" />

после (уже используется DNS)
<img width="360" height="50" alt="image" src="https://github.com/user-attachments/assets/800756bd-d7a6-4e3a-a571-ab52aeea9d7e" />

и включите в chrome "Использовать безопасный DNS-сервер". Выбираем поставика услуг DNS по умолчанию (так как мы уже сделали это на уровне системы)
<img width="675" height="158" alt="image" src="https://github.com/user-attachments/assets/268dc113-fe53-48fd-ad6a-e497f1de87a8" />

## 🌐 Проверка защиты
Перед началом настройки рекомендуется проверить текущее состояние:
- **DNS Leak Test**: https://dnsleaktest.com
если ваш ISP соотвествует вашему подключенному DNS, то все успешно работает.
ISP (Internet Service Provider) — это провайдер, который дает тебе интернет. (билайн, мтс и прочее)
<img width="137" height="66" alt="image" src="https://github.com/user-attachments/assets/cb1bc68e-daab-4599-a07c-f31ab9c80553" />
- **WebRTC Leak Test**: https://browserleaks.com/webrtc
если WebRTC Leak Test	✔ No Leak, то никаких утечек нету.
<img width="262" height="42" alt="image" src="https://github.com/user-attachments/assets/f2a6eb63-1125-4926-8673-4bb58cc3e6bf" />


⚠️ Что провайдер всё ещё видит даже с DoH?

- Твой IP-адрес

- Факт подключения к серверам Cloudflare

- Объём трафика (сколько данных скачал/отправил)

- Время твоего подключения

Но он не видит конкретные сайты, которые ты посещаешь.

♦️ **IP-адрес — это как твой почтовый адрес в интернете. Без него пакеты данных не знают, куда идти. Скрыть его полностью нельзя, но можно маскировать или шифровать**

## ⚠️ Что такое WebRTC и зачем его контролировать?
WebRTC — технология для видео-звонков прямо в браузере (например, Discord в браузере, Zoom через сайт, Telegram Web). Но она может слить твой локальный IP-адрес (адрес в твоей домашней сети), если её не приручить.

## 🛡️ Способы отключения/контроля WebRTC

### 🔧 В браузере (рекомендуемый способ)

**Firefox:**
1. Откройте `about:config` в адресной строке
2. Найдите параметр `media.peerconnection.enabled`
3. Переключите значение на **`false`**

**Chrome/Edge/Brave:**
1. Установите расширение [WebRTC Leak Prevent](https://chrome.google.com/webstore/detail/webrtc-leak-prevent/eiadekoaikejlgdbkbdfeijglgfdalml)
2. В настройках расширения выберите **«Disable non-proxied UDP»**

### ⚙️ Дополнительные методы защиты

**Блокировка через файл hosts:**
Добавьте следующие строки в файл `hosts` (расположен в `C:\Windows\System32\drivers\etc\hosts`):
```
0.0.0.0 stun.l.google.com
0.0.0.0 stun.services.mozilla.com
0.0.0.0 turn.services.mozilla.com
0.0.0.0 stun.server.com
```
*Примечание: Это не гарантирует 100% отключение, но может сломать работу WebRTC в некоторых браузерах.*

**Настройка фаервола:**
Можно заблокировать браузеры в фаерволе на доступ к портам, которые использует WebRTC:
- UDP порты: 3478, 3479, 5349, 5350, 19302, 19309
*Внимание: Это сложно и может сломать другие приложения.*

## 💡 Рекомендации
- Для большинства пользователей достаточно настройки в браузере
- Расширение WebRTC Leak Prevent обеспечивает хороший баланс между безопасностью и функциональностью
- Полное отключение WebRTC может нарушить работу видео-звонков в браузере
