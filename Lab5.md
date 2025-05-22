# Лабораторная работа 4: Контейнеры, cron-интеграция и systemd (Apache HTTP)

## 1. Развертывание Apache
### 1.1 Установка Podman
```bash
sudo dnf install -y podman
```

### 1.2 Запуск Apache

```bash
# загрузка apache
podman pull docker.io/library/httpd

# запуск контейнера
podman run -d --name apache_s -p 8080:80 --restart always httpd

# проверка работы
podman ps
curl http://localhost:8080

```

### 1.3 Изменение стартовой страницы
```bash
# кастомная страница
mkdir ~/apache-content
echo "<h1>Apache from 50nk31</h1><p>Date: $(date)</p>"> ~/apache-content/index.html

# перезапуск контейнера с корневой папкой
podman stop apache_s
podman rm apache_s
podman run -d --name apache_s -p 8080:80 -v ~/apache-content:/usr/local/apache2/htdocs --restart always httpd
```

### 1.4 Проверка перезапуска
```bash
# политика перезапуска
podman inspect apache_s | grep RestartPolicy

# создаем падение
podman stop apache_s && sleep 5 && podman ps
```

## 2. Настройка мониторинга диска и cron

### 2.1 Установка необходимых пакетов
```bash
sudo dnf install -y mailx postfix cronie
sudo systemctl enable --no2 postfix crond
```

### 2.2 Создание скрипта

```bash
sudo nano /usr/local/bin/disk_monitor.sh
```
 **Код скрипта**
 ```bash


```
