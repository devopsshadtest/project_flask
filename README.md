# CI/CD Pipeline for Python Flask App with Monitoring with Grafana(Prometheus)

## Огляд
Цей проєкт містить CI/CD пайплайн для Python Flask-додатку з Jenkins і Docker(Docker-compose), а також систему моніторингу з Prometheus і Grafana (+ cAdvisor).

## Вимоги до системи
- Jenkins з плагінами: Docker Pipeline, Git, Pipeline
- Docker і Docker Compose
- Python 3.12+
- Prometheus, Grafana, cAdvisor

## Структура репозиторія на гітхаб
- `app/` - код додатку на Flask
- `scripts/` - приклади скриптів автоматизації (backup, restart) - в пайплайні використовуємо аналог
- `.dockerignore` - виключення файлів для Docker
- `Dockerfile` - конфігурація multistage Docker-образу (size of image app ~70Mb vs 1Gb)
- `docker-compose.yml` - локальне розгортання з моніторингом
- `Jenkinsfile` - наш CI/CD пайплайн
- `prometheus.yml` - конфігурація для Prometheus
- `alerts.yml` - правила для алертів

## Процес розгортання CI/CD пайплайну
1. **Встановлення Jenkins**:
   - Встановіть Jenkins на локальну машину або VM.
Встановлення Jenkins
1. Перевірте залежності - java -version
Якщо Java не встановлена:
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
2. Додайте репозиторій Jenkins
Імпортуйте ключ GPG для репозиторію Jenkins:
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
Додайте репозиторій до джерел:
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
3. Встановіть Jenkins
sudo apt update
sudo apt install jenkins -y
4. Перевірте статус Jenkins
sudo systemctl status jenkins
Якщо служба активна (active/running), усе гаразд. (в даному проекті змінено порт у cAdvisor на 8085)

   - Додайте плагіни: Docker Pipeline, Git, Pipeline.
   - Налаштуйте доступ до Docker (`/var/run/docker.sock`).
2. **Конфігурація проєкту**:
   - Склонувати репозиторій з гітхабу.
   - У Jenkins створіть Pipeline проєкт із `Jenkinsfile`.
3. **Запуск**:
   - Запустіть пайплайн через Jenkins UI.
   - Етапи: Checkout → Stop Old Containers → Build App and Deploy with Docker Compose → Test Application → Restart App Container → Backup Images.

## Налаштування моніторингу:
1. **Запуск стеку**:
   - Виконайте `docker-compose up -d` для запуску додатку, Prometheus, Grafana та cAdvisor.
2. **Конфігурація Grafana**:
   - Відкрийте `http://localhost:3000` (логін: admin, пароль: admin можна не змінювати).
   - Додайте Prometheus як джерело даних (`http://prometheus:9090`). Алерти автоматично будуть завантажені також.
   - Створіть дашборд із метриками CPU та пам’яті.
   Метрики контейнерів (у нашому випадку це app):
Використання CPU(сумарне): sum(rate(container_cpu_usage_seconds_total{container_label_com_docker_compose_service="app"}[5m])).
Використання пам’яті: container_memory_usage_bytes{container_label_com_docker_compose_service="app"}.

3. **Налаштування алертів**:
   - У `alerts.yml` визначено два алерти:
     - `HighCPUUsage` (>80% CPU протягом 2 хвилин).
     - `HighMemoryUsage` (>500MB пам’яті протягом 2 хвилин).
   - Переглядайте алерти в Prometheus UI (`http://localhost:9090`).

## Використання скриптів автоматизації
1. **Бекап образу**: `./scripts/backup.sh`
2. **Рестарт контейнера**: `./scripts/restart.sh`

## Перевірка
- Додаток: `http://localhost:5000/api`
- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000`

##For stress testing:
1 - docker ps | grep app  
2 - docker exec -it 88e9bb06b3f2 apk add stress-ng   
3 - docker exec -it 88e9bb06b3f2 stress-ng --cpu 2 --timeout 300s