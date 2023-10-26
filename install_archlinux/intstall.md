## Загружаем установочный образ
## Подключение к WiFi:
 - `iwctl`
### Чтобы узнать название интерфейса пишем:
 - `device list`
### Поиск доступных сетей:
 - `device <i>устройство</i> scan`
<i>some text</i>

## Установка системы
```
pacstrap -K base linux linux-firmware iwd neovim dhcpcd
```

