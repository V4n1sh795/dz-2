# Отчёт по домашней работе

> Заполняйте отчёт по ходу выполнения задания. Для каждой найденной проблемы скопируйте секцию ниже и опишите не только внесённое изменение, но и ход диагностики.

## Проблема 1

### Что было обнаружено

Сервис не запускается

### Как проводилась диагностика

Укажите команды и действия, которыми вы получили информацию о проблеме, а также существенные результаты этих проверок.

```bash
sudo systemctl status homework-app.service
sudo journalctl -u homework-app
cat homework-app.service
ls -la /opt/linux-devops-homework/
```


### В чём была причина
В service файле приложения был указан не тот файл для запуска
### Что было изменено
Конфигурация homework-app.service 
```bash
+ ExecStart=/opt/linux-devops-homework/server.py
- ExecStart=/opt/linux-devops-homework/app.py
```


### Почему было выбрано это решение

Приложение не запускается, логов в журнале нет, ошибка ExecStart => само приложение не запускается, при проверке пути наличия файла, такгого файла не было


### Как проверялся результат

Изменился статус сервиса 
(code=exited, status=1/FAILURE)


```bash
sudo systemctl status homework-app.service
```


---
## Проблема 2

### Что было обнаружено

Опишите наблюдаемую ошибку или некорректное состояние системы.
(code=exited, status=1/FAILURE)
запуск падает с ошибкой
```log
Oct 04 14:14:31 mainpc server.py[12054]: PermissionError: [Errno 13] Permission denied: '/var/lib/linux-devops-homework/startup.log'
Oct 04 14:14:31 mainpc server.py[12054]:            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Oct 04 14:14:31 mainpc server.py[12054]:     return io.open(self, mode, buffering, encoding, errors, newline)
Oct 04 14:14:31 mainpc server.py[12054]:   File "/usr/lib/python3.12/pathlib.py", line 1015, in open
Oct 04 14:14:31 mainpc server.py[12054]:          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Oct 04 14:14:31 mainpc server.py[12054]:     with (STATE_DIR / 'startup.log').open('a', encoding='utf-8') as f:
Oct 04 14:14:31 mainpc server.py[12054]:   File "/opt/linux-devops-homework/server.py", line 18, in <module>
```
### Как проводилась диагностика

```bash
sudo systemctl status homework-app.service
sudo journalctl -r -u homework-app
cat homework-app.service
ls -la /opt/linux-devops-homework/
```


### В чём была причина
нет доступа на запись в папку
### Что было изменено
Конфигурация homework-app.service 
```bash
sudo sudo chown -R homework:homework /var/lib/linux-devops-homework
```


### Почему было выбрано это решение

Приложение не могла нормально писать логи, поэтому падало

### Как проверялся результат

Изменился статус сервиса 
 Active: active (running) since Sun 2026-10-04 14:18:52 +05; 5s ago


```bash
sudo systemctl status homework-app.service
```


---
## Проблема 3

### Что было обнаружено

Приложение не может писать в указанный лог файл


### Как проводилась диагностика

Укажите команды и действия, которыми вы получили информацию о проблеме, а также существенные результаты этих проверок.

```bash
sudo systemctl status homework-app.service
sudo journalctl -r -u homework-app
cat homework-app.service
ls -la /var/lib/linux-devops-homework/
```


### В чём была причина
Приложение не может писать в указанный лог файл
### Что было изменено
Права доступа на папку
```bash
sudo chown -R homework-app /var/lib/linux-devops-homework/
```


### Почему было выбрано это решение

PermissionError: [Errno 13] Permission denied: '/var/lib/linux-devo


### Как проверялся результат

Изменился статус сервиса 
active (running) since Wed 2026-10-07 21:12:03 +05; 4s ago

```bash
sudo systemctl status homework-app.service
```


---
## Проблема 4

### Что было обнаружено

Приложение слушает только localhost, игнорируя запросы из внешней сети


### Как проводилась диагностика

```bash
ss -tulnp
```


### В чём была причина
В конфиг файле было указано host = 127.0.0.1 
### Что было изменено
Конфигурация app.conf
```bash
- host = 127.0.0.1
+ host = 0.0.0.0
port = 8080
```
### Почему было выбрано это решение
Оно тут единственное, если не поднимать nginx
### Как проверялся результат

Изменился статус сервиса 
(code=exited, status=1/FAILURE)
0.0.0.0:8080                        0.0.0.0:*             users:(("python3",pid=4441,fd=3))

```bash
sudo ss -tulnp
```


---
## Итоговая проверка

```bash
sudo ./scripts/check.sh
```

```text
Linux DevOps Homework Checker

[PASS] systemd ExecStart points to the application
[PASS] service runs as homework:homework
[PASS] state directory ownership and permissions are correct
[PASS] service is enabled
[PASS] service is active
[PASS] running process has the expected UID
[PASS] TCP/8080 listens on 0.0.0.0
[PASS] GET /health returns the expected response
[PASS] fixed systemd unit is saved in the repository
[PASS] fixed application configuration is saved in the repository

Result: 10 passed, 0 failed
```
