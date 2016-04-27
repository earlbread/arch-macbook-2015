# 맥북프로 15인치 2015년형에서 Arch linux 설치하기

맥북프로 15인치 2015년형에 Arch linux를 설치하는 과정. 현재기준 최신버전인 2016.04.01 이미지(Kernel 4.5.1)를 사용하였다.

## 참고
 - [Arch Linux on MacBook Pro Retina 2014 with DM-Crypt, LVM and suspend to disk]
(http://loicpefferkorn.net/2015/01/arch-linux-on-macbook-pro-retina-2014-with-dm-crypt-lvm-and-suspend-to-disk/)
 - [Arch Linux guide: the always up-to-date Arch Linux tutorial](http://www.linuxveda.com/2016/02/08/the-always-up-to-date-guide-to-install-arch-linux/)

## Base Install
### 1. OSX에서 Arch linux를 위한 파티션 분할
펌웨어 업데이트와 OS X 사용 용도로 디스크 유틸리티에서 80G를 할당하고 나머지 파티션을 새로 생성한다.
### 2. Arch linux live usb 만들기
아치리눅스 다운로드 페이지(https://www.archlinux.org/download/)에서 이미지를 다운받아 부팅 usb를 생성.
### 3. Arch linux live usb로 부팅
부팅 usb를 연결 후 재부팅, alt 키를 누르고 있으면 선택 메뉴가 나오고 여기서 생성한 usb를 선택.
### 4. Font 설정
레티나의 영향으로 폰트가 작으므로 눈을 위해 폰트를 키운다.
```
setfont sun12x22
```
### 5. 네트워크 연결
커널 버전이 올라가서인지 예전과는 다르게 드라이버 설치 없이 와이파이 연결이 가능.
iplink로 와이파이 디바이스를 확인후 와이파이 연결. 여기서는 wlp3s0
```
wifi-menu -o wlp3s0
```
### 6. Arch linux를 위한 파티션 생성
OSX 에서 나눈 파티션을 삭제 후 리눅스에서 필요한 파티션을 생성.
여기서는 root(swapfile을 포함)와 home.
```
cgdisk /dev/sda
```
에서 직접 선택 혹은
```
sgdisk -d 4 /dev/sda
sgdisk -n 4::+40G /dev/sda
sgdisk -c 4:root /dev/sda
sgdisk -n 5:: /dev/sda
sgdisk -c 5:home /dev/sda
```
### 7. Arch linux 파티션 포맷
```
mkfs.ext4 /dev/sda4
mkfs.ext4 /dev/sda5
```
### 8. Arch linux 파티션 마운트
EFS 파티션을 부트 파티션으로 사용
```
mount /dev/sda4 /mnt
mkdir /mnt/home
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
mount /dev/sda5 /mnt/home
```
### 9. base install
```
pacstrap /mnt base base-devel
```
### 10. fstab 파일 생성
UUID로 fstab 생성
```
genfstab -U /mnt >> /mnt/etc/fstab
```
### 11. 기본사항 설정
#### 1. 설정들을 위해 /mnt로 root 변경
```
arch-chroot /mnt /bin/bash
```
#### 2. 호스트 네임 설정
```
echo earl > /etc/hostname
```
#### 3. Timezone 설정
```
ln -sf /usr/share/zoneinfo/Canada/Pacific /etc/localtime
hwclock --systohc --utc
```
#### 4. locale 설정
/etc/locale.gen 에서 en_US.UTF-8 ko_KR.UTF-8을 uncomment 후
```
locale-gen
```
#### 5. user설정
root password 변경
```
passwd
```
user 추가
```
useradd -m -G wheel,users -s /bin/bash earl
passwd earl
```
visudo 에서
```
%wheel ALL=(ALL) ALL
```
uncomment
### 12. 부트로더 설정
initramfs 생성
```
mkinitcpio -p linux
```
EFS 파티션에 systemd-boot 설치
```
bootctl --path=/boot install
```
/boot/loader/loader.conf
```
default arch
timeout 4
```
/boot/loader/entries/arch.conf
```
title     Arch
linux     /vmlinuz-linux
initrd    /initramfs-linux.img
options   root=UUID=<uuid> rw
```
UUID는 fstab 참조
### 13. 와이파이 도구 설치
재부팅하면 wifi-menu가 없기 때문에 설치해준다.
```
pacman -Sy wpa_supplicant dialog
```
### 14. Reboot
```
exit
reboot
```



## Graphic Interface 설정 및 Desktop 환경 설치
### 1. X-server 설치
```
sudo pacman -Sy xorg-server xorg-server-utils
```
mesa-libgl, xf86-input-libinput을 선택

### 2. xf86-input-libinput 설정
/etc/X11/xorg.conf.d/90-libinput.conf
```
Section "InputClass"
    Identifier "libinput touchpad catchall"
    MatchIsTouchpad "on"
    MatchDevicePath "/dev/input/event*"
    Driver "libinput"
    Option "NaturalScrolling" "true"
EndSection
```
### 3. Gnome 설치
```
sudo pacman -S gnome gnome-extra
```
libx264 선택

```
pacman -Sy guake
```
Gnome 설치 후 locale과 관련한 버그가 있는지 Gnome terminal이 실행 되지 않으므로 대체 터미널 설치.

```
systemctl enable gdm.service
systemctl enable NetworkManager
reboot
```
gdm을 enable 시키고 Gnome 실행 이후 네트워크 연결을 위해 NetworkManager를 부팅 시 실행시킨다.

## Post Installation
### 1. yaourt install
공식 저장소에서 지원하지 않는 패키지들을 설치하기 위해 yaourt를 설치한다.
```
git clone https://aur.archlinux.org/package-query.git
cd package-query
makepkg –sri
cd ..

git clone https://aur.archlinux.org/yaourt.git
makepkg -si
cd ..

```

### 2. 한글 폰트 설치
Arch를 처음 설치하면 한글 폰트가 설치되어 있지 않아 한글이 모두 깨져서 나온다.
한국어 설정(https://wiki.archlinux.org/index.php/Internationalization/Korean)을 참조하여 폰트를 설치하자.
현재 위키에 나온 폰트 이외에도 스포카에서 개발한 [spoqa han sans](https://aur.archlinux.org/packages/spoqa-han-sans/)가 이용 가능하다.
여기서는 나눔고딕과 나눔고딕 코딩 폰트를 설치
```
yaourt -Sy ttf-nanum ttf-nanum_coding
```

### 3. 한글 입력
ibus-hangul 설치
```
pacman -Sy ibus-hangul
```

ibus-setup 실행, input method에서 Korean-Hangul 추가
```
ibus-setup
```

.bashrc에 추가
```
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
```

ibus-setup-hangul 에서 필요한 설정
```
ibus-setup-hangul
```

Settings -> Region & Language -> Input Sources 에서 Korean(Hangul)추가 후 재부팅

### 4. Gnome Terminal 실행 안되는 문제 해결
```
localectl set-locale LANG="en_US.UTF-8"
sudo reboot
```

## 기타 필요사항
### 1. zsh 설치
```
sudo pacman -Sy zsh zsh-completions
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
chsh -s /usr/bin/zsh
```

### 2. Chrome 설치
```
sudo pacman -Sy chromium
```

### 3. locate 설치
우분투에서는 기본으로 설치되어 있는 locate를 설치해야한다.
```
sudo pacman -Sy mlocate
sudo updatedb
``` 
