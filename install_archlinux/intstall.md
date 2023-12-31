# Установка Arch Linux
## Подготовка
### Подключение к WiFi:
Запускаем iwctl:  
 - `iwctl`
##### Чтобы узнать название интерфейса пишем:
 - `device list`
##### Поиск доступных сетей:
 - `device устройство scan`  
> **_NOTE:_** _после выполнения ничего не выведет, но начнет сканировать сети_
##### Поучаем список сетей и подключаемся к нужной:
 - `device устройство get-networks`
 - `device устройство connect SSID`  
 > **_NOTE:_** <i>Если сеть скрытая, подключаемся коммандой </i> `device <устройство> connect-hidden <SSID>`  
##### После этого можно проверить подключение командой:
 - `ping google.com`
 
 ---

### Разметка дисков:
##### Смотрим какие диски есть в системе:
 - `fdisk -l`

На диске должны быть разделы:
 - корневой каталог `/`
 - для загрузки в UEFI нужен системный раздел UEFI
##### Пример схемы для UEFI с GPT
|Точка монтирования|Раздел|Размер|
|------------------|------|------|
|`/mnt/boot/efi`|`/dev/<системный_раздел_efi>`|Минимум 300 Миб|
|[SWAP]|`/dev/<раздел_подкачки>`|Минимум 500МиБ|
|`/mnt`|`/dev/корневой_раздел`|Все что осталось|
---
### Форматирование разделов:
После создания разделов форматируем их.  
Например чтобы отформатировать корневой раздел в `ext4`:  
 - `mkfs.ext4 /dev/<корневой_раздел>`  
Включаем swap:  
 - `mkswap /dev/<раздел_подкачки>`  
Если вы создали системный раздел EFI, отформатируйте его в FAT32 с помощью mkfs.fat.  
 > **_ВАЖНО:_** Выполняйте форматирование, только если вы создали новый раздел в процессе разметки. Если системный раздел EFI уже существует, его форматирование уничтожит загрузчики других установленных операционных систем.
 - `mkfs.fat -F 32 /dev/<системный_раздел_efi>`
---
### Монтирование дисков:  
Создаем `boot` и `home` в `mnt`:
 ```bash
mkdir /mnt/{boot,home}  
mkdir /mnt/boot/efi
 ```

Mонтируем корневой раздел в папку `/mnt`:
 - `mount /dev/<корневой_раздел> /mnt`

Монтируем `efi` и `home`:
 ```bash
 mount /dev/<домашняя_папка> /mnt/home
 mount /dev/<системный_раздел_efi> /mnt/boot/efi
 ```
---
### Установка зеркал:  
В файле `/etc/pacman.d/mirrorlist` добавляем в самом начале строку:
 - `Server = https://mirror.yandex.ru/archlinux/$repo/os/$arch`
---
### Установка базовых пакетов
```bash
pacstrap -K base linux linux-firmware iwd neovim dhcpcd grub xdg-user-dirs git curl zsh openssh
```

## Установка системы:
### Генерация fstab:
 - `genfstab -U /mnt >> /mnt/etc/fstab`
### Chroot:
Вход в новую систему:
 - `arch-chroot /mnt`
### Установка часового пояса:
 - `ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime`
 - `hwclock --systohc`
### Настройка сети:
Задаем имя хоста:
 - `echo "<имя_хоста>" >> /etc/hostname`
### Установка пароля суперпользователя:
 - `passwd`
### Установка загрузчика:
##### В режиме UEFI:
 - `grub-install`
##### Настройка установленного загрузчика:
Самый простой вариант:
 - `grub-mkconfig`

> **_ВАЖНО:_** Полученый файл не следует редактировать вручную, так как автоматическая конфигурация достаточно сложная и запутанная

<details>
<summary>Более сложный вариант с ручной конфигурацией:</summary>
Вынос конфигурации grub в отдельный файл, и защита от изменений grub.cfg:

 ```bash
 echo ". $prefix/menu.cfg" >> /boot/grub/grub.cfg
 chattr +i /boot/grub/grub.cfg
 echo "#grub menu config" >> /boot/grub/menu.cfg
 ```

Содержимое файла `menu.cfg`:
 - `nvim /boot/grub/menu.cfg`
 ```bash
 set timeout=5
 menuentry "Arch Linux" {
 linux /boot/vmlinuz-linux root=UUID=<UUID_root_диска> rw
 initrd /boot/initramfs-linux.img
 }
 ```
Это самая простая конфигурация, другие параметры можно посмотреть [тут](https://wiki.archlinux.org/title/GRUB) 
</details>

##### Добавляем автостарт интернета:
```
systemctl enable dchpcd
systemctl enable iwd
```
## Перезагрузка:
Вводим `exit`

Размонтируем все диски:
 - `umount -R /mnt`

Перезагружаем:
 - `systemctl reboot`
