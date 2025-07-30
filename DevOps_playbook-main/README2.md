# 🚀 Полное руководство по установке Kubernetes кластера 



---

## 🎯 Что вы получите в итоге?

После запуска этого проекта у вас будет:

### 🏗️ **Полноценный Kubernetes кластер**
- **3 master-сервера** (управляющие ноды) для высокой надежности
- **2 worker-сервера** (рабочие ноды) для запуска приложений
- **Автоматическое восстановление** при сбоях

### 📊 **Система мониторинга**
- **Prometheus** — собирает данные о работе системы
- **Grafana** — красивые графики и дашборды
- **Alertmanager** — отправляет уведомления о проблемах

### 📝 **Система логирования**
- **Loki** — собирает все логи в одном месте
- **Grafana** — просмотр и поиск по логам

### 🌐 **Сетевые сервисы**
- **NGINX Ingress Controller** — маршрутизация веб-трафика
- **MetalLB** — балансировка нагрузки для bare metal

### 🔄 **GitOps платформа**
- **ArgoCD** — управление приложениями через Git
- **Автоматическое развертывание** при изменениях в коде

### 💾 **Система хранения**
- **Local Path Provisioner** — хранение данных на дисках серверов

---

## 🖥️ Что нужно для запуска?

### На вашем компьютере (откуда запускаете):
- **Linux** (Ubuntu 20.04+, CentOS 8+, или аналогичный)
- **Python 3.6+** (язык программирования)
- **pip** (менеджер пакетов Python)
- **git** (для скачивания проекта)

### На серверах (куда устанавливаете):
- **Ubuntu 20.04+** или **CentOS 8+**
- **Доступ по SSH** (удаленное подключение)
- **Пользователь с правами sudo** (или root)
- **Минимум 2 ГБ RAM** на каждый сервер
- **Минимум 20 ГБ свободного места** на диске

---


---

## 🚀 Пошаговая инструкция 

### Шаг 1: Подготовка вашего компьютера

#### 1.1 Установите Python и pip
```bash
# Проверьте, установлен ли Python
python3 --version

# Если не установлен, установите:
sudo apt update
sudo apt install python3 python3-pip
```

#### 1.2 Установите git
```bash
# Проверьте, установлен ли git
git --version

# Если не установлен, установите:
sudo apt install git
```

### Шаг 2: Скачивание проекта

```bash
# Скачайте проект
git clone <URL_ВАШЕГО_РЕПОЗИТОРИЯ>
cd vs2

# Проверьте, что файлы скачались
ls -la
```

**Что происходит:** Мы скачиваем все файлы проекта на ваш компьютер.

### Шаг 3: Установка Ansible

```bash
# Установите Ansible
pip install ansible

# Проверьте установку
ansible --version
```


### Шаг 4: Установка дополнительных модулей

```bash
# Перейдите в папку с проектом
cd Ansible/Playbooks/RKE2

# Установите необходимые коллекции
ansible-galaxy collection install -r collections/requirements.yaml
```

**Что происходит:** Мы устанавливаем дополнительные модули для работы с Kubernetes и Linux.

### Шаг 5: Подготовка серверов

#### 5.1 Создайте пользователя на всех серверах
На каждом сервере выполните:
```bash
# Создайте пользователя ansible
sudo adduser ansible

# Добавьте пользователя в группу sudo
sudo usermod -aG sudo ansible

# Настройте SSH ключи (рекомендуется)
ssh-copy-id ansible@IP_АДРЕС_СЕРВЕРА
```

#### 5.2 Проверьте подключение
```bash
# Попробуйте подключиться к серверу
ssh ansible@192.168.1.10

# Если работает, выйдите
exit
```

### Шаг 6: Настройка списка серверов

Откройте файл `inventory/hosts.ini` в любом текстовом редакторе:

```ini
[servers]
server1 ansible_host=192.168.1.10
server2 ansible_host=192.168.1.11
server3 ansible_host=192.168.1.12

[agents]
agent1 ansible_host=192.168.1.13
agent2 ansible_host=192.168.1.14

[rke2:children]
servers
agents

[rke2:vars]
ansible_user=ansible
```


### Шаг 7: Настройка переменных

Откройте файл `inventory/group_vars/all.yaml`:

```yaml
# Сетевые настройки
vip: 192.168.1.50                    # Виртуальный IP для кластера
lb_range: 192.168.1.80-192.168.1.90  # Диапазон IP для балансировщика

# Версии компонентов
metallb_version: v0.13.12

# Доменные имена
domain_name: "local"

# Мониторинг
prometheus_retention: "7d"
grafana_admin_password: "admin123"

# ArgoCD
argocd_admin_password: "admin123"
```

**Важно:** Измените IP-адреса на ваши!

### Шаг 8: Проверка подключения

```bash
# Проверьте, что Ansible может подключиться ко всем серверам
ansible -i inventory/hosts.ini all -m ping
```

**Ожидаемый результат:**
```
server1 | SUCCESS => {"changed": false, "ping": "pong"}
server2 | SUCCESS => {"changed": false, "ping": "pong"}
server3 | SUCCESS => {"changed": false, "ping": "pong"}
agent1 | SUCCESS => {"changed": false, "ping": "pong"}
agent2 | SUCCESS => {"changed": false, "ping": "pong"}
```

### Шаг 9: Запуск установки

```bash
# Запустите основной сценарий
ansible-playbook -i inventory/hosts.ini site.yaml
```

**Что происходит:**
1. Ansible подключается к каждому серверу
2. Устанавливает необходимые пакеты
3. Скачивает и устанавливает RKE2
4. Настраивает кластер
5. Устанавливает все дополнительные сервисы

**Время выполнения:** 15-30 минут

---

## 🔍 Что происходит во время установки?

### Этап 1: Подготовка серверов (5-10 минут)
- ✅ Обновление системы
- ✅ Установка необходимых пакетов
- ✅ Настройка сетевых параметров

### Этап 2: Установка RKE2 (10-15 минут)
- ✅ Скачивание RKE2
- ✅ Настройка конфигурационных файлов
- ✅ Запуск первого master-сервера

### Этап 3: Расширение кластера (5-10 минут)
- ✅ Добавление дополнительных master-нод
- ✅ Добавление worker-нод
- ✅ Настройка высокой доступности

### Этап 4: Установка инфраструктуры (5-10 минут)
- ✅ MetalLB (балансировщик нагрузки)
- ✅ Local Path Provisioner (хранение)
- ✅ NGINX Ingress Controller

### Этап 5: Установка мониторинга (5-10 минут)
- ✅ Prometheus (сбор метрик)
- ✅ Grafana (визуализация)
- ✅ Loki (логирование)

### Этап 6: Установка GitOps (5 минут)
- ✅ ArgoCD для управления приложениями

---

## ✅ Как проверить, что все работает?

### 1. Проверка нод кластера

```bash
# Скопируйте конфигурацию с первого сервера
scp ansible@192.168.1.10:/etc/rancher/rke2/rke2.yaml ~/kubeconfig

# Настройте переменную окружения
export KUBECONFIG=~/kubeconfig

# Проверьте ноды
kubectl get nodes
```

**Ожидаемый результат:**
```
NAME      STATUS   ROLES                  AGE   VERSION
server1   Ready    control-plane,master   10m   v1.29.4
server2   Ready    control-plane,master   10m   v1.29.4
server3   Ready    control-plane,master   10m   v1.29.4
agent1    Ready    <none>                 8m    v1.29.4
agent2    Ready    <none>                 8m    v1.29.4
```

### 2. Проверка подов

```bash
# Посмотрите все поды в кластере
kubectl get pods -A
```

**Ожидаемый результат:** Много подов в статусе "Running"

### 3. Проверка сервисов

```bash
# Посмотрите все сервисы
kubectl get svc -A
```

---

## 🌐 Доступ к веб-интерфейсам

После установки вы сможете зайти в веб-интерфейсы:

### Grafana (мониторинг)
- **URL:** http://grafana.local
- **Логин:** admin
- **Пароль:** admin123

### Prometheus (метрики)
- **URL:** http://prometheus.local
- **Логин:** не требуется

### ArgoCD (GitOps)
- **URL:** http://argocd.local
- **Логин:** admin
- **Пароль:** admin123

**Важно:** Добавьте записи в файл `/etc/hosts`:
```
192.168.1.50 grafana.local
192.168.1.50 prometheus.local
192.168.1.50 argocd.local
```

---

## 🛠️ Частые проблемы и их решения

### ❌ Проблема: "Permission denied" при SSH
**Решение:**
```bash
# Проверьте права на SSH ключи
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub

# Проверьте подключение
ssh ansible@192.168.1.10
```

### ❌ Проблема: "No space left on device"
**Решение:**
```bash
# Проверьте свободное место
df -h

# Очистите кэш пакетов
sudo apt clean  # для Ubuntu
sudo yum clean all  # для CentOS
```

### ❌ Проблема: "Connection refused"
**Решение:**
```bash
# Проверьте, что сервер доступен
ping 192.168.1.10

# Проверьте SSH порт
telnet 192.168.1.10 22
```

### ❌ Проблема: "kubectl command not found"
**Решение:**
```bash
# Установите kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

---

## 📚 Полезные команды

### Основные команды kubectl:

```bash
# Посмотреть все ноды
kubectl get nodes

# Посмотреть все поды
kubectl get pods -A

# Посмотреть все сервисы
kubectl get svc -A

# Посмотреть все namespace
kubectl get namespaces

# Посмотреть логи пода
kubectl logs -n monitoring prometheus-kube-prometheus-prometheus-0

# Зайти в контейнер
kubectl exec -it -n monitoring prometheus-kube-prometheus-prometheus-0 -- /bin/sh
```

### Команды для мониторинга:

```bash
# Посмотреть метрики Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# Доступ к Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

### Команды для ArgoCD:

```bash
# Доступ к ArgoCD UI
kubectl port-forward -n argocd svc/argocd-server 8080:80

# Получить пароль ArgoCD
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

## 🔒 Безопасность 

### Обязательные действия:

1. **Измените пароли по умолчанию:**
   ```yaml
   grafana_admin_password: "ВАШ_СЛОЖНЫЙ_ПАРОЛЬ"
   argocd_admin_password: "ВАШ_СЛОЖНЫЙ_ПАРОЛЬ"
   ```

2. **Настройте файрвол:**
   ```bash
   # Установите ufw
   sudo apt install ufw

   # Разрешите SSH
   sudo ufw allow ssh

   # Разрешите Kubernetes порты
   sudo ufw allow 6443
   sudo ufw allow 10250

   # Включите файрвол
   sudo ufw enable
   ```



## 🤝 Получение помощи

### Если что-то не работает:

1. **Проверьте логи:**
   ```bash
   # Логи RKE2 сервера
   sudo journalctl -u rke2-server -f

   # Логи RKE2 агента
   sudo journalctl -u rke2-agent -f
   ```

2. **Проверьте статус сервисов:**
   ```bash
   # Статус RKE2
   sudo systemctl status rke2-server
   sudo systemctl status rke2-agent
   ```

3. **Проверьте сеть:**
   ```bash
   # Проверьте порты
   netstat -tlnp | grep 6443
   ```

### Полезные ресурсы:
- [Документация Kubernetes](https://kubernetes.io/docs/)
- [Документация RKE2](https://docs.rke2.io/)
- [Документация Ansible](https://docs.ansible.com/)

---

## 🎉 Поздравляем!

Вы успешно установили production-ready Kubernetes кластер! Теперь у вас есть:

- ✅ **Высокодоступный кластер** с несколькими серверами
- ✅ **Система мониторинга** для отслеживания работы
- ✅ **Централизованное логирование** для анализа проблем
- ✅ **GitOps платформа** для автоматического развертывания
- ✅ **Готовность к production** сбалансированная нагрузка

