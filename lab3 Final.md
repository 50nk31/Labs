# Лабораторная работа №3: Файловые системы, LVM и RAID

## 1. Создание виртуальных дисков и RAID1

### 1.1 Создание двух виртуальных дисков по 500мб
**Команды** 
```bash
sudo dd if=/dev/zero of=/root/disk1.img bs=1M count=500
sudo dd if=/dev/zero of=/root/disk2.img bs=1M count=500
```

### 1.2 Подключение дисков как loop-устройств
```bash
sudo losetup -f /root/disk1.img
sudo losetup -f /root/disk2.img
```

### 1.3 Создание RAID1
```bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/loop0 /dev/loop1
```

### 1.4 Проверка состояния (RAID)
```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

## 2. Создание LVM

### 2.1 Инициализация RAID как физический том
```bash
sudo pvcreate /dev/md0
```

### 2.2 Создание группы томов
```bash
sudo vgcreate vg_lab /dev/md0
```

### 2.3 Создание логических томов
```bash
sudo lvcreate -L 300M -n lv_data vg_lab
sudo lvcreate -l 100%FREE -n lv_backup vg_lab
```

### 2.4 Форматирование и монтирование
```bash
sudo mkfx.ext4 /dev/vg_lab/lv_data
sudo mkfx.xfs /dev/vg_lab/lv_backup

sudo mkdir -p /srv/data
sudo mkdir -p /srv/backup
sudo mount /dev/vg_lab/lv_data /srv/data
sudo mount /dev/vg_lab/lv_backup /srv/backup
```

## 3. Создание Snapshot и расширение тома

### 3.1 Snapshot для lv_backup
```bash
sudo lvcreate -L 50M -s -n lv_backup_snap /dev/vg_lab/lv_backup
```

### 3.2 Увеличиваем lv_data на 100МБ
```bash
sudo resize2fs /dev/vg_lab/lv_data

```
## 4. Имитация отказа и восстановление RAID

### 4.1 "Ломаю" один из дисков
```bash
sudo mdadm /dev/md0 --fail /dev/loop0
```

### 4.2 Проверка RAID (по сценарию должно быть **degraded**)
```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

### 4.3 Возварт диска в RAID
```bash
sudo mdadm /dev/md0 --re-add /dev/loop0
```

## Скриншот-отчет
**Результат команды**
![cat /proc/mdstat](https://i.imgur.com/P6IieyK.jpeg)

**Результат команды**
![lvdisplay](https://i.imgur.com/PGGcw4d.jpeg)

**Результат команды**
![Размеры томов](https://i.imgur.com/rJ4DAir.jpeg)

**Состояние RAID после отказа**
![RAID](https://i.imgur.com/GhKibdt.jpeg)

