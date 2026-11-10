# Домашнее задание к занятию 16 «Платформа мониторинга Sentry»
# Развёртывание Self-Hosted Sentry на виртуальной машине в Yandex Cloud

## Введение

В рамках выполнения домашнего задания к занятию 16 «Платформа мониторинга Sentry» было выполнено развёртывание Self-Hosted Sentry на виртуальной машине в облаке Yandex Cloud. Вместо использования Free Cloud account (как предлагалось в задании) был выбран вариант с самостоятельным развёртыванием, так как доступ к внешним сервисам был ограничен.

## 1. Требования к ресурсам

Self-Hosted Sentry — требовательная к ресурсам система. Согласно официальной документации (https://develop.sentry.dev/self-hosted/), минимальные требования для профиля `errors-only` (только мониторинг ошибок):
- **CPU:** 2 ядра
- **RAM:** 7 ГБ
- **Диск:** 20 ГБ свободного места

Для полноценной работы (профиль `feature-complete`, который используется по умолчанию) требуется:
- **CPU:** 4 ядра
- **RAM:** 14–16 ГБ
- **Диск:** 30–50 ГБ

**Важно:** при попытке установки на ВМ с 4 ГБ RAM установщик завершится с ошибкой `Insufficient memory`. Поэтому была создана ВМ с **15 ГБ RAM** и диском на **35 ГБ**.

Также перед установкой необходимо увеличить `vm.max_map_count` (требование Elasticsearch, который используется внутри Sentry):

```bash
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```
# Клонируем репозиторий
git clone https://github.com/getsentry/self-hosted.git

# Переходим в папку
cd self-hosted

# Запускаем установщик
./install.sh

# Настройка доступа по внешнему IP-адресу (решение ошибки CSRF)
После запуска Sentry при попытке входа по адресу http://<IP>:9000 возникала ошибка You do not have permission to load this data (или csrf_failure в логах). Это стандартная защита Django (на котором основан Sentry): запрос с внешнего IP блокируется, если этот IP не указан в списке доверенных источников.

В логах docker compose logs web было видно:

csrf_failure (reason='Origin checking failed - http://51.250.66.80:9000 does not match any trusted origins.')
Решение: явно указать внешний IP в двух файлах конфигурации.

# Файл sentry/config.yml
Раскомментировать (или добавить) строку system.url-prefix с внешним IP и портом:
yaml
system.url-prefix: 'http://51.250.66.80:9000'

# Файл sentry/sentry.conf.py
В самом низу файла, в блоке # CSRF Settings, добавить:

python
# CSRF_TRUSTED_ORIGINS = ["http://51.250.66.80:9000", "http://127.0.0.1:9000"]
Критически важно: протокол в CSRF_TRUSTED_ORIGINS должен точно совпадать с тем, по которому вы заходите в браузере. Если Sentry работает без SSL (по HTTP), то и в списке должен быть http://, а не https://. В первой попытке был указан https://, из-за чего ошибка сохранялась.

# Применение настроек
После правки файлов необходимо пересоздать контейнеры web и nginx:

bash
docker compose up -d --force-recreate web nginx

#Скрины по дз
<img width="974" height="741" alt="image" src="https://github.com/user-attachments/assets/8133215d-1b04-4dc9-afe6-915460144a5d" />
<img width="974" height="644" alt="image" src="https://github.com/user-attachments/assets/12d6e6b4-3c53-4f12-99a4-29d0fd6643c5" />
<img width="974" height="592" alt="image" src="https://github.com/user-attachments/assets/49b7ae22-5f65-4a09-981a-357ace67c0c3" />
<img width="974" height="596" alt="image" src="https://github.com/user-attachments/assets/0586f01b-1131-4756-a244-900c429795ef" />
<img width="974" height="574" alt="image" src="https://github.com/user-attachments/assets/6421c9d6-5c44-4c2b-9566-163eb399ff5e" />
<img width="974" height="582" alt="image" src="https://github.com/user-attachments/assets/23600250-816f-4f25-99b0-f2e45011ba21" />
<img width="974" height="501" alt="image" src="https://github.com/user-attachments/assets/e95ea3ed-2029-4fa7-a085-2a458f6ff689" />
<img width="974" height="501" alt="image" src="https://github.com/user-attachments/assets/5b0915d3-a111-4e69-8b7f-851165c2453e" />
<img width="974" height="611" alt="image" src="https://github.com/user-attachments/assets/e0736f55-9e7b-4489-a95a-89dbf0c5b6bc" />
<img width="974" height="745" alt="image" src="https://github.com/user-attachments/assets/bf3d0df1-8063-4a90-a96a-f6dd0337e145" />
<img width="974" height="745" alt="image" src="https://github.com/user-attachments/assets/71cf78f9-9604-4e9f-9891-fa5ce22d66be" />










