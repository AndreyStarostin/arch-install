# Руководство по установке Arch linux с KDE Plasma (UEFI)

##  Содержание
  * [**Начало установки**](#начало-установки)
  * [**Разметка диска**](#разметка-диска)
    * [UEFI](#разметка-диска-uefi)
  * [**Установка базовой системы**](#установка-базовой-системы)
    * [Обновление зеркал](#обновление-зеркал-с-помощью-reflector)
    * [Установка основных пакетов](#установка-основных-пакетов)
    * [Создание fstab](#создание-fstab)
  * [**Chroot**](#chroot)
    * [Активация swap](#активация-swap)
    * [Установка даты и времени](#установка-даты-и-времени)
    * [Установка языка](#установка-языка)
    * [Настройка имени хоста](#настройка-имени-хоста)
    * [Network Manager](#установка-networkmanager)
    * [Задать пароль root](#задать-пароль-root)
    * [Установка GRUB Bootloader](#установка-grub-bootloader)
      * [Настройка GRUB (UEFI)](#настройка-grub-uefi)
    * [Завершение установки](#завершение-установки)
  * [**Загрузка установленной системы**](#загрузка-установленной-системы)
    * [Добавить нового пользователя](#добавить-нового-пользователя)
    * [Настройка sudo для группы wheel](#настройка-sudo-для-группы-wheel)
  * [**Войти под созданным пользователем**](#войти-в-систему-под-новым-пользователем)
    * [Проверить подключение к Интернету](#проверить-подключение-к-интернету-1)
    * [Проверка наличия обновлений](#проверка-наличия-обновлений)
    * [Драйверы Xorg и GPU](#драйверы-xorg-и-gpu)
    * [Включение репозитория Multilib](#включение-репозитория-multilib)
    * [Библиотеки MESA (32bit)](#библиотеки-mesa-32bit)
    * [Установка экранного менеджера SDDM](#установка-sddm)
    * [Установка KDE Plasma и основных приложений](##установка-kde-plasma-и-основных-приложений)
    * [Аудио-утилиты и Bluetooth](#audio-утилиты-и-bluetooth)
    * [Мои приложения](#мои-приложения)
    * [Приложения для работы](#приложения-для-работы)
  * [**Заключение**](#заключение)
  * [**Дополнительно**](#дополнительно-необязательно)
    * [Yay](#yay)
    * [Zsh](#установка-zsh)
    * [Установка и настройка Oh My Zsh](#установка-и-настройка-oh-my-zsh)
    * [Поддержка принтеров](#поддержка-принтеров)
    * [PipeWire](#pipewire)
    * [Paccache](#paccache)
    * [Docker](#docker)
    * [kubectl](#kubectl)
    
</br>

## Начало установки
- Необходимо **[загрузить ISO образ](https://www.archlinux.org/download/)** и записать его на пустой USB-накопитель.
- После того, как образ будет записан, загрузиться в Arch Live Environment и выполнить следующие действия:

### Загрузка раскладок
Для получения списка всех доступных раскладок используйте команду:
```
localectl list-keymaps
```

Чтобы найти раскладку клавиатуры, используйте следующую команду, заменив `[search_term]` кодом для вашего языка, страны или раскладки:
```
localectl list-keymaps | grep -i [search_term]
```

### Теперь загрузите раскладку (ru)
```
loadkeys [keymap]
```

### Проверить подключение к Интернету
```
ping -c 4 google.com
```
- Если вы подключены через Ethernet, то ваш интернет будет работать из коробки.
- Если вы используете Wi-Fi, используйте «iwctl» для подключения к локальной сети.
```
iwctl
[iwd]# device list
[iwd]# station DEVICE scan
[iwd]# station DEVICE get-networks
[iwd]# station DEVICE connect SSID
```
- Если этот шаг будет успешным, мы перейдем к следующему.

### Обновление системных часов
```
timedatectl set-ntp true
```
</br>


## Разметка диска
> :warning: Будьте предельно осторожны при управлении своими дисками, если вы удалите свои данные, то НЕ вините меня. Выберите тип разбиения диска (используйте UEFI или MBR, в зависимости от вашей системы).

###  Разметка диска (UEFI)
Мы собираемся создать три раздела на нашем жестком диске, «EFI BOOT & SWAP & ROOT», используя «fdisk».
Просмотреть список доступных дисков можно при помощи команды:
```
fdisk -l
```
Выберите нужный диск:
```
fdisk /dev/[disk name]
```
- [disk name] = выберите нужный диск из вывода команды `fdisk -l` или `lsblk`.

Чтобы создать новую таблицу разделов и удалить всю текущую информацию о разделах, введите команду `g` для создания таблицы разделов GUID (GPT). Пропустите этот шаг, если необходимая таблица уже была создана.

На выбранном накопителе должны присутствовать следующие разделы:
  - Раздел для загрузки в режиме UEFI /boot.
  - Раздел swap
  - Раздел для корневого каталога /.
  
Создание UEFI раздела 
```
n = Новый раздел
Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-500118158, default 2048): Нажать enter
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-500118158, default 500117503): +512M
Created a new partition 1 of type 'Linux file system' and of size 512 MiB.

t = Задать тип раздела
Command (m for help): t
Для просмотра всех типов разделов ввести команду: "L". 1 = EFI System.
Partition type or alias (type L to list all): 1
Changed type of partition 'Linux filesystem' to 'EFI System'
```
Создание SWAP раздела, размер зависит от количества оперативной памяти
```
n = Новый раздел
Command (m for help): n
Partition number (2-128, default 2): 2
First sector (1050624-500118158, default 2048): Нажать enter
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1050624-500118158, default 500117503): +8G
Created a new partition 1 of type 'Linux file system' and of size 512 MiB.

t = Задать тип раздела
Command (m for help): t
Partition number(1,2, default 2): 2
Для просмотра всех типов разделов ввести команду: "L". 19 = Linux swap.
Partition type or alias (type L to list all): 19
Changed type of partition 'Linux filesystem' to 'Linux swap'
```
Создание корневого раздела
```
n = Новый раздел
Command (m for help): n
Partition number (3-128, default 3): 3
First sector (17827840-500118158, default 2048): Нажать enter
Last sector, +/-sectors or +/-size{K,M,G,T,P} (17827840-500118158, default 500117503): Нажать enter
Created a new partition 1 of type 'Linux file system' and of size 512 MiB.

t = Задать тип раздела
Command (m for help): t
Partition number(1-3, default 3): 3
Для просмотра всех типов разделов ввести команду: "L". 20 = Linux filesystem.
Partition type or alias (type L to list all): 19
Changed type of partition 'Linux filesystem' to 'Linux filesystem'
```
Сохранение изменений и выход
```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disk.
```
### Форматирование разделов (UEFI)
```
mkfs.fat -F32 /dev/[efi partition name]
mkfs.ext4 /dev/[root partiton name] 
```

### Монтировнаие разделов (UEFI)
```
mount /dev/[root partition name] /mnt
mkdir -p /mnt/boot/efi
mount /dev/[efi partition name] /mnt/boot/efi
```

## Установка базовой системы
###  Обновление зеркал с помощью Reflector
```
reflector -c County1 -c Country2 -a 12 -p https --sort rate --save /etc/pacman.d/mirrorlist
```
Замените «County1» и «County2» на страны рядом с вами или на страну, в которой вы живете. См. **[ Reflector ](https://wiki.archlinux.org/index.php/reflector)** для большей информации. Замените https на http в случае необходимости.
###  Установка основных пакетов
```
pacstrap /mnt base base-devel linux linux-firmware linux-headers nano intel-ucode reflector mtools dosfstools efibootmgr
```
- Замените `linux` на *linux-hardened* , *linux-lts* или *linux-zen* , чтобы установить ядро ​​по вашему выбору.
- Замените `linux-headers` на тип ядра по вашему выбору соответственно (например, если вы установили `linux-zen` , вам понадобится `linux-zen-headers`).
- Замените `nano` редактором по вашему выбору (например , `vim` или `vi` ).
- Замените `intel-ucode` на `amd-ucode` , если вы используете процессор AMD.

###  Создание fstab
(используйте ключ -U или -L, чтобы для идентификации разделов использовались UUID или метки, соответственно)
```
genfstab -U /mnt >> /mnt/etc/fstab
```
Проверьте получившийся файл `/mnt/etc/fstab` и отредактируйте его в случае ошибок.
</br>
## Chroot
```
arch-chroot /mnt
```
### Активация swap
```
mkswap /dev/[swap partition name]
Скопировать UUID
swapon /dev/[swap partition name]
Отредактировать fstab
nano /etc/fstab
Добавить строки:
# /dev/nvme0n1p2
UUID=[Вставить скопированный UUID]       none    swap    defaults        0       0
```
### Установка даты и времени
```
ln -sf /usr/share/zoneinfo/Регион/Город /etc/localtime
hwclock --systohc
```
Замените «Регион» и «Город» в соответствии с вашим часовым поясом. Дополнительную информацию см. в разделе **[ Часовой пояс ](https://wiki.archlinux.org/index.php/installation_guide#Time_zone)**.
###  Установка языка
Здесь мы будем использовать `ru_RU.UTF-8` , но если вы хотите установить свой язык, то раскомментируйте свою локаль.

####  Изменить locale.gen
```
nano /etc/locale.gen
```
Раскомментируйте строку ниже
```
#ru_RU.UTF-8 UTF-8
```
сохранить и выйти.

####  Создать локаль
```
locale-gen
```

####  Добавить LANG в locale.conf
```
echo LANG=ru_RU.UTF-8 > /etc/locale.conf
```

####  Добавляем раскладки в vconsole
```
nano /etc/vconsole.conf
KEYMAP=[keymap]
FONT=cyr-sun16
```
###  Настройка имени хоста
```
echo arch > /etc/hostname
```
Замените `arch` на свое имя.

#### Отредактировать Hosts
```
nano /etc/hosts
```
##### Добавьте в файл следующие строки
```
127.0.0.1    localhost
::1          localhost
127.0.1.1    arch.localdomain arch
```
Замените `arch` на свое имя.
Сохранить и выйти.
### Установка NetworkManager
```
pacman -S networkmanager
systemctl enable NetworkManager
```
### Задать пароль root
```
passwd
```
### Установка GRUB Bootloader
```
pacman -S grub
```
#### Настройка GRUB (UEFI)
```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id='Arch Linux'
```
##### Создание конфигурационного файла grub.cfg
```
grub-mkconfig -o /boot/grub/grub.cfg
```
### Завершение установки
```
exit 
umount -a
reboot
```
</br>

## Загрузка установленной системы

### Войти под пользователем root

### Добавить нового пользователя
```
useradd -mG wheel [username]
```
Замените `[username]` именем своего пользователя.

#### Установите пароль пользователя `[username]`
```
passwd [username]
```

### Настройка sudo для группы wheel
```
EDITOR=nano visudo
```

#### Найдите и раскомментируйте следующую строку
```
#%wheel ALL=(ALL) ALL
```
Сохранить и выйти.

### Выйти из ROOT
```
exit
```
## Войти в систему под новым пользователем
### Проверить подключение к Интернету
```
ping -c 4 google.com
```
- Если вы подключены через Ethernet, то ваш интернет будет работать из коробки.
- Если вы используете Wi-Fi, используйте утилиту «nmcli» из ранее устанорвленного пакета NetworkManager для подключения к локальной сети.
```
nmcli --ask dev wifi connect [network-ssid]
```
- [network-ssid] = Имя Wi-Fi сети
- Далее утилита попросит ввести пароль

### Проверка наличия обновлений
```
sudo pacman -Syu
```
###  Драйверы Xorg и GPU
```
sudo pacman -S xorg [xf86-video-ваш тип GPU]
```
- Для графических процессоров Nvidia введите «nvidia» и «nvidia-settings » . Для получения дополнительной информации/старых графических процессоров см. [ Arch Wiki - Nvidia ](https://wiki.archlinux.org/index.php/NVIDIA).
- Для более новых графических процессоров AMD введите `xf86-video-amdgpu`.
- Для устаревших графических процессоров Radeon, таких как HD серии 7xxx и ниже, введите `xf86-video-ati`.
- Для выделенной графики Intel введите `xf86-video-intel`.

### Включение репозитория Multilib (необязательно)
multilib содержит 32-разрядное программное обеспечение и библиотеки, которые можно использовать для запуска и сборки 32-разрядных приложений на 64-разрядных установках (например, [ Wine ](https://www.winehq.org/), [ Steam ](https:/ /store.steampowered.com/) и т. д.).
Отредактируйте `/etc/pacman.conf` раскомментируйте следующие строки.
```
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```

### Библиотеки MESA (32bit)
Этот пакет требуется Steam, если вы играете в игры, используя Vulkan Backend.
```
sudo pacman -S lib32-mesa
```

### Установка SDDM
```
sudo pacman -S sddm
sudo systemctl enable sddm
```
### Установка KDE Plasma и основных приложений
```
sudo pacman -S plasma konsole dolphin ark kate kcalc krunner partitionmanager packagekit-qt5 plasma-nm plasma-pa
```
Пакеты | Описание
--------- | ----------
plasma | Окружение рабочего стола KDE Plasma.
konsole | Терминал KDE.
dolphin | Файловый менеджер.
ark | Архиватор.
kate | Текстовый редактор.
kcalc | Калькулятор.
krunner | Быстрый поиск и запуск приложений.
partitionmanager | Диспетчер дисков и разделов KDE.
packagekit-qt5 | Привязки Qt5 для PackageKit.
plasma-nm | Настройка громкости KDE.
plasma-pa | Настройка сети KDE.
openssh | ssh сервер.

#### Включить службу OpenSSH
```
sudo systemctl enable --now sshd.service
```
### Audio утилиты и Bluetooth
```
sudo pacman -S alsa-utils bluez bluez-utils pulseaudio-bluetooth
```
Пакеты | Описание
--------- | ----------
alsa-utils | Содержит (помимо других утилит) утилиты alsamixer и amixer.
bluez | Предоставляет стек протоколов Bluetooth.
bluez-utils | Предоставляет утилиту bluetoothctl.
pulseaudio-bluetooth | Дополнительный модуль для подключения аудио через блютуз.

####  Включить службу Bluetooth
```
sudo systemctl enable bluetooth.service
```

###  Мои приложения
Вы можете установить все следующие пакеты или только тот, который вам нужен.
#### Установка через pacman
```
sudo pacman -S chromium openssh qbittorrent audacious screen wget git neofetch plasma-browser-integration sddm-kcm telegram-desktop
```
#### Установка через yay
```
yay -S skypeforlinux-stable-bin
```
Пакеты | Описание
--------- | ----------
chromium | Веб-браузер.
plasma-browser-integration | Интеграция веб-браузеров с рабочим столом Plasma.
qbittorrent | BitTorrent-клиент на основе Qt.
audacious  | Музыкальный проигрыватель на основе Qt.
wget | Wget — бесплатная утилита для неинтерактивной загрузки файлов из Интернета.
screen  | Позволяет переключаться между терминалами, в которых выполняются процессы
git | Утилиты командной строки Github.
neofetch | Инструмент системной информации из командной строки.
sddm-kcm | Модуль KConfig (KCM), который интегрируется в системные настройки KDE и служит для настройки SDDM
telegram-desktop | Telegram мессенджер
skypeforlinux-stable-bin | Skype мессенджер

### Приложения для работы

#### Установка через pacman
```
sudo pacman -S libreoffice-still okular remmina libvncserver freerdp virtualbox virtualbox-host-dkms pycharm-community-edition
```
#### Установка через yay
```
yay -S anydesk-bin teamviewer
```
Пакеты | Описание
--------- | ----------
libreoffice-still | LibreOffice — мощный офисный пакет (стабильная ветвь обновлений).
okular | Просмотрщик документов.
anydesk-bin | Быстрое приложение для удаленного рабочего стола.
teamviewer | Программное обеспечение для удаленной поддержки.
pycharm-community-edition | Python IDE для разработчиков.
remmina | Remmina — клиент удалённого рабочего стола.
libvncserver | Remmina — поддержка VNC.
freerdp | Remmina — поддержка RDP.
virtualbox | VirtualBox — гипервизор, предназначенный для запуска виртуальных машин, поверх существующей операционной системы.
virtualbox-host-dkms | DKMS-хук Pacman автоматически перекомпилирует модули virtualbox-host-dkms при каждом обновлении VirtualBox и/или ядра.




###  Заключение
Теперь все установлено, и после перезагрузки вы попадете на экран входа в систему с графическим интерфейсом. Вы также можете выполнить несколько дополнительных шагов, упомянутых ниже, чтобы еще больше улучшить свою систему.

##  Дополнительно (необязательно)
- Yay предоставит ваши пакеты из AUR (пользовательский репозиторий), которые недоступны в официальном репозитории.
- Paccache можно использовать с чистыми кэшированными пакетами pacman как вручную, так и автоматически.
- Cups поддержка установки и настройки принтеров
###  Установка [ Yay ](https://github.com/Jguer/yay)
Установщик пакетов из поьзовательского репозитория.
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
### Установка [Zsh](https://wiki.archlinux.org/index.php/zsh/) 
Одна из современных командных оболочек UNIX, использующаяся непосредственно как интерактивная оболочка, либо как скриптовый интерпретатор.
```
sudo pacman -S zsh zsh-completions
```

####  Изменение вашей оболочки
Сначала проверьте текущую оболочку, запустив:
```
echo $SHELL
```

####  Чтобы получить список всех доступных/установленных SHELL:
```
chsh -l
```

####  Установка Zsh в качестве нашей оболочки по умолчанию.
```
chsh -s /usr/bin/zsh
```
Чтобы изменения вступили в силу, нужно будет выйти из системы и снова войти в систему или, еще лучше, выполнить «перезагрузку».

### Установка и настройка [Oh My Zsh](https://ohmyz.sh/) 
Oh My Zsh — это управляемая сообществом платформа с открытым исходным кодом для управления вашей конфигурацией Zsh.
```
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```
####  Скачать тему [ Powerlevel10k ](https://github.com/romkatv/powerlevel10k/) для Oh My Zsh
```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
#### Скачать рекомендуемые шрифты
```
yay -S ttf-dejavu ttf-meslo-nerd-font-powerlevel10k
```
Также установите шрифт терминала Konsole на `MesloGS-NF-Regular`.

#### Установите Powerlevel10k в качестве темы Zsh
```
nano ~/.zshrc
```
Найдите строку, начинающуюся с `ZSH_THEME="...."` , и замените имя темы, чтобы строка теперь выглядела так: `ZSH_THEME="powerlevel10k/powerlevel10k"` Теперь выполните `source ~/.zshrc`.

#### Настройка
> ***Для новых пользователей*** при первом запуске мастер настройки Powerlevel10k задаст вам несколько вопросов. Если он не запускается автоматически, введите «p10k configure». Мастер настройки создает `~/.p10k.zsh` на основе ваших предпочтений. Дополнительную настройку можно выполнить, отредактировав этот файл. Он содержит множество комментариев, которые помогут вам ориентироваться в параметрах конфигурации.

### Поддержка принтеров
```
sudo pacman -S cups print-manager system-config-printer
```
Пакеты | Описание
--------- | ----------
cups | Стандартная система печати с открытым исходным кодом.
print-manager | Менеджер принтеров.
system-config-printer | Настройки для принтеров.

#### Запуск сервера печати
```
sudo systemctl enable --now cups
```

### PipeWire
[PipeWire](https://wiki.archlinux.org/title/PipeWire) это новый мультимедийный фреймворк низкого уровня. Он призван обеспечить захват и воспроизведение как аудио, так и видео с минимальной задержкой и поддержкой приложений на основе PulseAudio, JACK, ALSA и GStreamer.
#### Установка
```
sudo pacman -S pipewire
```

### Paccache
Очистка кэша pacman

#### Установка
```
sudo pacman -S pacman-contrib
```
#### Узнать количество и размер кэшированных пакеров
```
sudo ls /var/cache/pacman/pkg/ | wc -l
du -sh /var/cache/pacman/pkg/
```
#### Очистка кэша вручную
```
sudo paccache -rk 3
```
k указывает на сохранение кол-во версии каждого пакета в кэше.

#### Автоматическая очмистка кэша
Создайте файл в `/etc/pacman.d/hooks`
```
sudo mkdir /etc/pacman.d/hooks
sudo nano /etc/pacman.d/hooks/clean_cache.hook
```
Добавьте в него следующие строки
```
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *
[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -r
```
Сохранить и выйти.

### Docker
#### Установка
```
sudo pacman -S docker
```
#### Запуск службы
```
sudo systemctl enable --now docker
```
#### Настройка
```
sudo groupadd docker
sudo usermod -aG docker $USER
```
Каждый пользователь в группе docker имеет права, равноценные правам суперпользователя.

### Kubectl
#### Установка
Загрузить последнюю версию:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```
Для загрузки определенной версии (1.25.0), используйте следующую команду:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.25.0/bin/linux/amd64/kubectl
```
Сделать двоичный файл kubectl исполняемым:
```
chmod +x ./kubectl
```
Переместить двоичный файл в директорию из переменной окружения PATH:
```
sudo mv ./kubectl /usr/local/bin/kubectl
```
Проверка работоспособности:
```
kubectl version --client
```
#### Настройка автодополнения
Скрипт дополнения ввода kubectl для Zsh может быть сгенерирован с помощью команды:
```
kubectl completion zsh
```
Чтобы подключить его во все сессии командной оболочки, добавьте следующую строчку в файл ```~/.zshrc```:
```
source <(kubectl completion zsh)
```
Добавить алиас "k" для команды kubectl:
```
echo 'alias k=kubectl' >>~/.zshrc
echo 'compdef __start_kubectl k' >>~/.zshrc
```
Если появляется такая ошибка как ```complete:13: command not found: compdef```, то добавьте следующее содержимое в начало вашего файла ```~/.zshrc```:
```
autoload -Uz compinit
compinit
```
