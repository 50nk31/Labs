# Лабораторная работ №5: SELinux, безопаность и аудит
**Вся работа выполнялась через терминал чтобы была видна последовательность действий**

## 1. Проверка SELinux
```bash
# режим enforcing
sudo setenforce 1

# текущий статус selinux
sestatus

# текущая политика
getenforce
```

*постоянное сохранение режима*
```bash
sudo sed -i 's/SELINUX=permissive/SELINUX=enforcing/g' /etc/selinux/config
```

## 2. Диагностика SELinux и устранение блокировок
**Создал тестовый каталог и файлы:**
```bash
sudo mkdir -p /srv/webtest
sudo echo "Test Page" | sudo tee /srv/webtest/index.html
```

**Настройка тест-сервера:**

```bash
# apache
sudo tee /etc/httpd/conf.d/webtest.conf <<EOF
<VirtualHost *:80>
  DocumentRoot /srv/webtest
  ServerName webtest.local
</VirtualHost>
EOF

# запуск сервера
sudo systemctl start httpd
sudo systemctl enable httpd
```
**Проверка доступа и анализ логов:**

```bash
curl http://localhost

# логи selinux
sudo ausearch -m avc -ts recent # в них ничего не было

# альтернативный метод просмотра логов
sudo grep "avc:.*denied" /var/log/audit/audit.log
```

**Генерация и применение политки:**
```bash
sudo grep httpd /var/log/audit/audit.log | audit2allow -M httpd_custom

sudo semodule -i httpd_custom.pp

sudo semodule -l | grep httpd_custom
```

## 3. Создание модуля SELinux

**Создание файла:**
```bash
sudo tee mycustom.te <<EOF
module mycustom 1.0;

require {
  type unconfined_t;
  type bin_t;
  class file { execute read open };
}

allow unconfined_t bin_t:file { execute read open };
EOF
```

**Компиляция и установка модуля:**
```bash
checkmodule -M -m -o mycustom.mod mycustom.te
semodule_package -o mycustom.pp -m mycustom.mod

sudo semodule -i mycustom.pp

sudo semodule -l | grep mycustom
```
## 4. Аудит системных изменений
