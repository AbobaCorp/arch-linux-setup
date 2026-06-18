![Логотип Arch для светлой темы](img/archlinux-logo-black.svg#gh-light-mode-only)
![Логотип Arch для темной темы](img/archlinux-logo-white.svg#gh-dark-mode-only)

# Установка Arch Linux

Пошаговая инструкция по установке и базовой настройке Arch Linux.

---

## Шаг 1. Подключение к Интернету

* **Ethernet:** Ничего делать не требуется, подключение настроится автоматически.
* **Wi-Fi:** Используем утилиту `iwctl`.
  ```bash
  iwctl
  station list
  station <адаптер> get-networks
  station <адаптер> connect "SSID_сети"
  # Ждем 10-15 секунд
  quit
  ```
  > 💡 **Совет:** Если возникла проблема с подключением к Wi-Fi, попробуйте выполнить команду `rfkill unblock all` и повторите попытку.

---

## Шаг 2. Настройка пакетного менеджера

1. Откройте список зеркал:
   ```bash
   nano /etc/pacman.d/mirrorlist
   ```
   Проверьте, что `reflector` сгенерировал список зеркал. Если нет — закомментируйте все и раскомментируйте те, которыми хотите пользоваться.

2. Настройте `pacman` для параллельной загрузки:
   ```bash
   nano /etc/pacman.conf
   ```
   Раскомментируйте параметр `ParallelDownloads` и задайте желаемое значение (например, `5`).

---

## Шаг 3. Разметка диска

Найдите нужный диск:
```bash
lsblk
```
Запустите утилиту разметки:
```bash
cfdisk /dev/<диск>
```

**Вариант 1: Без выделения `/home` в отдельный раздел**
| № | Тип | Размер | Точка монтирования |
|---|---|---|---|
| 1 | EFI system | 256M | `/boot/efi` |
| 2 | Linux filesystem | Все свободное место | `/` |

**Вариант 2: С выделением `/home`**
| № | Тип | Размер | Точка монтирования |
|---|---|---|---|
| 1 | EFI system | 256M | `/boot/efi` |
| 2 | Linux filesystem | 25G ~ 30G | `/` |
| 3 | Linux filesystem | Все свободное место | `/home` |

---

## Шаг 4. Форматирование разделов

```bash
# Форматируем EFI раздел
mkfs.vfat /dev/<диск1>

# Форматируем корневой раздел
mkfs.ext4 /dev/<диск2>

# (Если /home выносили в отдельный раздел)
mkfs.ext4 /dev/<диск3>
```

---

## Шаг 5. Монтирование разделов

```bash
# Монтируем корень
mount /dev/<диск2> /mnt

# Создаем директорию и монтируем EFI
mkdir -p /mnt/boot/efi
mount /dev/<диск1> /mnt/boot/efi

# (Если /home выносили в отдельный раздел)
mkdir -p /mnt/home
mount /dev/<диск3> /mnt/home
```

---

## Шаг 6. Установка пакетов

Ниже приведен список необходимых пакетов. Выберите по одному варианту из категорий «Дисплейный менеджер» и «Окружение рабочего стола».

| Категория | Пакеты |
|---|---|
| **Необходимый минимум** | `base base-devel linux linux-firmware linux-headers nano vim bash-completion grub efibootmgr` |
| **Дисплейный сервер** | `xorg` |
| **Шрифты** | `ttf-ubuntu-font-family ttf-hack ttf-dejavu ttf-opensans` |
| **Дисплейный менеджер** (выберите ОДИН) | `sddm` / `lightdm` / `lxdm` / `gdm` |
| **Окружение рабочего стола** (выберите ОДНО) | `gnome` / `plasma` / `cinnamon` / `budgie` / `xfce4` / `lxqt` / `lxde` |
| **Драйверы** (при необходимости) | `nvidia` (проприетарный драйвер NVIDIA) |
| **Open-source драйверы** | `mesa intel-media-driver libva-intel-driver vulkan-intel vulkan-nouveau vulkan-radeon xf86-video-amdgpu xf86-video-ati xf86-video-nouveau` |

**Пример команды установки (для KDE Plasma и SDDM):**
```bash
pacstrap /mnt base base-devel linux linux-firmware linux-headers nano vim bash-completion grub efibootmgr xorg ttf-ubuntu-font-family ttf-hack ttf-dejavu ttf-opensans sddm plasma
```

**Установка всех open-source драйверов:**
```bash
pacstrap /mnt mesa intel-media-driver libva-intel-driver vulkan-intel vulkan-nouveau vulkan-radeon xf86-video-amdgpu xf86-video-ati xf86-video-nouveau
```

---

## Шаг 7. Генерация fstab

```bash
genfstab /mnt >> /mnt/etc/fstab
```

---

## Шаг 8. Смена корня системы (chroot)

Переходим в установленную систему:
```bash
arch-chroot /mnt
```

---

## Шаг 9. Включение сервисов

```bash
# Включаем NetworkManager
systemctl enable NetworkManager

# Включаем дисплейный менеджер (sddm / lxdm / gdm / lightdm)
systemctl enable <название_DM>
```

---

## Шаг 10. Пользователи и пароли

```bash
# Создаем пользователя
useradd -m <имя_пользователя>

# Устанавливаем пароль пользователю
passwd <имя_пользователя>

# Устанавливаем пароль пользователю root
passwd
```

**Настройка прав `sudo`:**
```bash
visudo
```
> ⚠️ **Важно:** Рекомендуется использовать `visudo`. Если вы не умеете работать в `vi(m)`, можно отредактировать файл напрямую через `nano /etc/sudoers`, но убедитесь, что права доступа не сломаются. Найдите строку `root ALL=(ALL:ALL) ALL` и добавьте ниже: `<ваш_пользователь> ALL=(ALL:ALL) ALL`.

---

## Шаг 11. Локали и язык системы

1. Раскомментируйте нужные локали:
   ```bash
   nano /etc/locale.gen
   ```
   Пример:
   ```text
   en_US.UTF-8 UTF-8
   ru_RU.UTF-8 UTF-8
   ```
2. Укажите желаемый язык системы:
   ```bash
   nano /etc/locale.conf
   ```
   Впишите строку:
   ```text
   LANG="en_US.UTF-8"
   ```
3. Сгенерируйте локали:
   ```bash
   locale-gen
   ```

---

## Шаг 12. Установка загрузчика (GRUB)

```bash
# Устанавливаем GRUB на диск (без номера раздела!)
grub-install /dev/<диск>

# Убираем параметр quiet для вывода логов при загрузке
nano /etc/default/grub
# В параметре GRUB_CMDLINE_LINUX_DEFAULT уберите "quiet"

# Генерируем конфиг
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Шаг 13. Перезагрузка в новую систему

```bash
# Выходим из chroot в шелл Live CD
exit

# Размонтируем все разделы диска
umount -R /mnt

# Перезагружаемся
reboot
```
> ⚠️ **Не забудьте** отключить установочную флешку в BIOS/UEFI перед загрузкой.

---

## Шаг 14. Подключение к Интернету (в установленной системе)

* **Ethernet:** Ничего делать не требуется.
* **Wi-Fi:** В большинстве DE предусмотрен GUI для подключения. Если система спросит, подключаться ли через GUI — **«Да»** Подключаемся как обычно; **«Нет»**
  Используйте `nmtui` (Text User Interface для NetworkManager):
  ```bash
  nmtui
  ```
  1. Выберите пункт **Radio**, проверьте, что Wi-Fi включен. Если нет — включите.
  2. Вернитесь назад, выберите **Activate a connection**.
  3. Выберите вашу сеть (AP), введите пароль и подключитесь.
  4. Выйдите из утилиты.

---

## Шаг 15. Настройка даты и времени

```bash
# Устанавливаем часовой пояс (относительный путь из /usr/share/zoneinfo)
sudo timedatectl set-timezone Europe/Moscow

# Включаем NTP (синхронизация времени по сети)
sudo timedatectl set-ntp true
```

---

## Шаг 16. Создание swap-файла

```bash
# Создаем swap-файл (например, размером 6G)
sudo mkswap -U clear --size 6G --file /swapfile

# Активируем swap
sudo swapon /swapfile
```

Добавьте запись в `fstab` для автоматического монтирования:
```bash
nano /etc/fstab
```
Допишите в конец файла (колонки разделяйте **Tab**'ом):
```text
/swapfile  none  swap  defaults  0  0
```
