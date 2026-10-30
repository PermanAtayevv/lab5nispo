# Лабораторная работа №5

Проектирование модели управления версиями ПО, конфигурациями и непрерывной поставкой.

**Автор:** Атаев Перман, 4 курс, группа 13, БГУ, ФПМИ.  
**Вариант:** задания 1–2.

## Отчёт

- [Отчёт в Markdown](docs/report.md)
- [Отчёт в PDF](docs/report.pdf)
- [Все 27 скриншотов](docs/screenshots/)
- [Журналы выполнения](docs/logs/)

## Результаты

Terraform: создание контейнера Nginx, изменение порта 8085 → 8086, удаление ресурсов, входная переменная и outputs. Итоговый план без изменений.

Ansible: подключение к двум узлам kind, zsh 5.9, файл с информацией об узле, Nginx на worker, handler и HTTP-проверки. Повторные запуски setup.yml и web.yml: changed=0, failed=0.

## Особенности стенда

В ЛР №3 использовались Flask и Jaeger в Docker Compose. Для этой работы создан отдельный kind-кластер lab5 с control-plane и worker. Ansible использует community.docker.docker и пользователя root внутри контейнеров. Это адаптация требования о виртуальных машинах из ЛР №3; согласование с преподавателем не подтверждено. Порт 8088 Nginx доступен внутри worker, не опубликован в Windows.

## Файлы

- terraform/main.tf, variables.tf, outputs.tf — конфигурация Windows/Docker Desktop.
- terraform/.terraform.lock.hcl — фактически созданный lock-файл провайдера.
- ansible/kind-config.yaml — конфигурация двух узлов.
- ansible/inventory.ini — группы и параметры подключения.
- ansible/setup.yml — пакеты и информация об узлах.
- ansible/web.yml — Nginx, handler и проверки.

## Запуск Terraform (PowerShell)

```powershell
cd terraform
terraform init
terraform validate
terraform plan -var="container_name=lab5-group13-nginx"
terraform apply -var="container_name=lab5-group13-nginx"
terraform output
```

Nginx: http://localhost:8086. Параметр -var передаётся при каждом plan/apply. Конфигурация использует Windows named pipe; для Linux её нужно адаптировать.

Удаление учебных ресурсов:

```powershell
terraform destroy -var="container_name=lab5-group13-nginx"
```

При keep_locally=true образ остаётся в Docker.

## Запуск Ansible (Ubuntu WSL)

Требуются Docker Desktop с WSL Integration, доступ пользователя к Docker, kind, Ansible и коллекция community.docker. Команды выполняются из корня репозитория.

```bash
ansible-galaxy collection install community.docker
# Только если кластер lab5 ещё не существует:
kind create cluster --name lab5 --config ansible/kind-config.yaml --wait 5m
ansible k8s -i ansible/inventory.ini -m ansible.builtin.raw \
  -a 'test -x /usr/bin/python3 || (apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y python3)'
ansible k8s -i ansible/inventory.ini -m ansible.builtin.ping
ansible-playbook -i ansible/inventory.ini ansible/setup.yml --syntax-check
ansible-playbook -i ansible/inventory.ini ansible/setup.yml
ansible-playbook -i ansible/inventory.ini ansible/web.yml --syntax-check
ansible-playbook -i ansible/inventory.ini ansible/web.yml
```

Повторите оба playbook для проверки идемпотентности. При немедленном повторе ожидаются changed=0 и failed=0. После истечения cache_valid_time обновление APT-кеша может отражаться как изменение.

## Ограничения и статус сдачи

Все присланные скриншоты включены в отчёт, включая диагностику. Листинги отчёта восстановлены по выполненным инструкциям; исходные файлы и журналы находятся в рабочем репозитории автора. Недоступное руководство «Ansible, Playbook» из методички не удалось сверить; выполнение всех его примеров не подтверждено. Commit, push и Pull Request на момент подготовки отчёта не подтверждены.

Не добавляйте в Git .terraform/, *.tfstate*, приватные ключи и kubeconfig с учётными данными. Файл .terraform.lock.hcl следует сохранить.

## Репозиторий

https://github.com/PermanAtayevv/lab5nispo
