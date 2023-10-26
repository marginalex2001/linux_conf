## Загружаем установочный образ
## Подключение к WiFi:
 - ```iwctl``
### Чтобы узнать название интерфейса пишем:
 - ```dewice list```
### Поиск доступных сетей:
 - ```device _устройство_ scan```

## Установка системы
```
pacstrap -K base linux linux-firmware iwd neovim dhcpcd
```

