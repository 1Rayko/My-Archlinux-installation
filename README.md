# My-Archlinux-installation

## Разметка диска

`bash 
gdisk /dev/sda`

Необходимо создать 3 раздела(efi под boot, swap, и основной раздел) 
Для создания раздела вводим: `n` 

300M выедляем под EFI. Код: `EF00`
2G или 4G выделяем под swap. Код: `8200`
Все остальное пространство под ФС. Код: `8300`

## Форматирование диска

Boot-раздел: `mkfs.fat -F32 /dev/sda1`
Swap: `mkswap -L swap /dev/sda2`
Включить swap: `swapon /dev/sda2`

## Создание тома и субволумов

```bash
mkfs.btrfs -L arch /dev/sda3 -f
mount /dev/sda3 /mnt
btrfs su cr /mnt/@
btrfs su cr /mnt/@var
btrfs su cr /mnt/@home
btrfs su cr /mnt/@snapshots
umount /mnt
mount -o noatime,compress=zstd,ssd,subvol=@ /dev/sda3 /mnt
mkdir -p /mnt/{home,boot,var,.snapshots}
mount -o noatime,compress=zstd,ssd,subvol=@var /dev/sda3 /mnt/var
mount -o noatime,compress=zstd,ssd,subvol=@home /dev/sda3 /mnt/home
mount -o noatime,compress=zstd,subvol=@snapshots /dev/sda3 /mnt/.snapshots
mount /dev/sda1 /mnt/boot
```


##  Наваливаем базу системы
`pacstrap /mnt base base-devel linux linux-firmware vim`

## Генерируем fstab
`genfstab -pU /mnt >> /mnt/etc/fstab`

## Chroot'имся и ставим root пароль

```bash
arch-chroot /mnt
passwd
```
## Настройка локалей, конфигов и прочего в etc
### Имя тостера (пк)
`vim /etc/hostname`

### Тайм-зона

`ln -sf /usr/share/zoneinfo/Asia/Krasnoyarsk /etc/localtime`

###  Локали

`vim /etc/locale.gen`

Раскоментить строку `ru.RU_UTF-8 UTF-8`

Сгенерировать локали
`locale-gen`


### Язык консоли, кириллица

`vim /etc/vconsole.conf`

прописываем: `KEYMAP=ru
FONT=cyr-sun16`

### Язык системы

`vim /etc/locale.conf`

Прописываем: `LANG="ru_RU.UTF-8"`

### Запуск пакмана
`pacman-key --init`

### Загрузка популярных ключей
`pacman-key --populate archlinux`

### multilib раскоментировать
`vim /etc/pacman.conf`


## Ставим софт
```bash
pacman -Sy ranger bash-completion openssh arch-install-scripts networkmanager sudo git wget htop neofetch xdg-user-dirs grub efibootmgr grub-btrfs os-prober xorg-server xorg-xinit xorg-drivers xf86-video-amdgpu pulseaudio pavucontrol obs-studio telegram-desktop firefox rofi kitty bspwm sxhkd feh code
```
## Что-то там с ядром короче

mkinitcpio -p linux

## Юзеры

`vim /etc/sudoers` - раскоментировать wheel после root

`useradd -m -g users -G wheel <имя пользователя>`
`passwd <имя пользователя>`

## Запуск сетевого менеджера

`systemctl enable NetworkManager.service`

## Ставим загрузчик
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
```

## Ребутимся
<Ctrl+D>

`reboot`

## Заходим от пользователя, даем права на запуск оконника; редактируем конфиги

```bash
mkdir ~/.config/{bspwm,sxhkd}
cp /usr/share/doc/bspwm/examples/bspwmrc ~/.config/bspwm/
cp /usr/share/doc/bspwm/examples/sxhkdrc ~/.config/sxhkd/
chmod +x ~/.config/bspwm/bspwmrc


echo ~/.xinitrc >> sxhkd& 
echo ~/.xinitrc > exec bspwm
```


# Дотфайлы
![Тут](https://github.com/zerocodex86/dotfiles)
