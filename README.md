# Домашнее задание к занятию «Система мониторинга Zabbix»

Выполнил: Нестеренко Андрей

---

## Задание 1. Установка Zabbix Server с веб-интерфейсом

В рамках задания был установлен Zabbix Server с веб-интерфейсом на Debian 11.

Используемые компоненты:

- Zabbix Server 6.0 LTS
- Zabbix Frontend
- PostgreSQL
- Apache
- Zabbix Agent

### Использованные команды

```bash
su -

apt update
apt upgrade -y

apt install -y postgresql postgresql-contrib

wget https://repo.zabbix.com/zabbix/6.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_6.0+debian11_all.deb
dpkg -i zabbix-release_latest_6.0+debian11_all.deb
apt update

apt install -y zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent

runuser -u postgres -- createuser --pwprompt zabbix
runuser -u postgres -- createdb -O zabbix zabbix

cd /tmp
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | runuser -u zabbix -- psql zabbix

nano /etc/zabbix/zabbix_server.conf
nano /etc/zabbix/apache.conf

systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2

systemctl status zabbix-server
systemctl status zabbix-agent
systemctl status apache2
```

### Настройки

В файле `/etc/zabbix/zabbix_server.conf` был указан пароль от базы данных:

```text
DBPassword=aa
```

В файле `/etc/zabbix/apache.conf` была указана временная зона:

```text
php_value date.timezone Europe/Moscow
```

### Скриншот авторизации/админки Zabbix

![Zabbix dashboard](images/zabbix-dashboard.png)

---

## Задание 2. Установка Zabbix Agent на два хоста

В рамках задания были настроены два хоста:

| Хост | IP-адрес | Роль |
|---|---:|---|
| Zabbix server | 192.168.229.135 | Zabbix Server + Zabbix Agent |
| zabbix-agent-2 | 192.168.229.136 | Zabbix Agent |

---

### Настройка агента на Zabbix Server

На сервере был отредактирован файл:

```bash
nano /etc/zabbix/zabbix_agentd.conf
```

Настройки агента:

```text
Server=127.0.0.1,192.168.229.135
ServerActive=192.168.229.135
Hostname=Zabbix server
```

После настройки агент был перезапущен:

```bash
systemctl restart zabbix-agent
systemctl enable zabbix-agent
systemctl status zabbix-agent
```

---

### Установка и настройка агента на втором хосте

На втором хосте `zabbix-agent-2` были выполнены команды:

```bash
su -

apt update

apt install -y wget curl gnupg2 lsb-release nano

wget https://repo.zabbix.com/zabbix/6.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_6.0+debian11_all.deb
dpkg -i zabbix-release_latest_6.0+debian11_all.deb
apt update

apt install -y zabbix-agent

nano /etc/zabbix/zabbix_agentd.conf
```

Настройки агента на втором хосте:

```text
Server=192.168.229.135
ServerActive=192.168.229.135
Hostname=zabbix-agent-2
```

После настройки агент был перезапущен:

```bash
systemctl restart zabbix-agent
systemctl enable zabbix-agent
systemctl status zabbix-agent
```

---

### Проверка доступности агентов

На Zabbix Server была установлена утилита `zabbix-get`:

```bash
apt install -y zabbix-get
```

Проверка агента на самом сервере:

```bash
zabbix_get -s 127.0.0.1 -k agent.ping
```

Проверка агента на втором хосте:

```bash
zabbix_get -s 192.168.229.136 -k agent.ping
```

Ожидаемый результат:

```text
1
```

---

### Скриншот раздела «Настройка → Узлы сети»

![Zabbix hosts](images/zabbix-hosts.png)

---

### Скриншот лога Zabbix Agent

![Zabbix agent log](images/zabbix-agent-log.png)

---

### Скриншот раздела «Мониторинг → Последние данные»

![Zabbix latest data](images/zabbix-latest-data.png)
