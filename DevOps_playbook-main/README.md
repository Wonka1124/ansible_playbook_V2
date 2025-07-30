# Автоматизированная установка Kubernetes кластера с RKE2

## 📋 Что это такое?

Этот проект представляет собой полную автоматизированную систему для развертывания production-ready Kubernetes кластера на базе RKE2 (Rancher Kubernetes Engine 2). Проект использует Ansible для автоматизации всех процессов установки и настройки.

### 🎯 Что вы получите в итоге:

- **Высокодоступный Kubernetes кластер** с несколькими master-нодами
- **Автоматический балансировщик нагрузки** (MetalLB)
- **Система мониторинга** (Prometheus + Grafana)
- **Централизованное логирование** (Loki)
- **GitOps платформа** (ArgoCD)
- **Ingress контроллер** (NGINX)
- **Система хранения** (Longhorn)

---

## 🏗️ Архитектура проекта

```
┌─────────────────────────────────────────────────────────────┐
│                    Ansible Playbooks                       │
├─────────────────────────────────────────────────────────────┤
│  📁 Ansible/Playbooks/RKE2/                              │
│  ├── 📄 site.yaml                    # Главный playbook   │
│  ├── 📁 inventory/                   # Конфигурация хостов│
│  ├── 📁 roles/                      # Роли для задач      │
│  └── 📁 collections/                # Зависимости         │
└─────────────────────────────────────────────────────────────┘
```

### 📂 Структура ролей:

- **prepare-nodes** - Подготовка серверов
- **rke2-download** - Скачивание RKE2
- **kube-vip** - Настройка виртуального IP
- **rke2-prepare** - Подготовка RKE2
- **add-server** - Добавление master-нод
- **add-agent** - Добавление worker-нод
- **apply-manifests** - Применение манифестов
- **storage** - Настройка системы хранения
- **ingress** - Настройка ingress контроллера
- **prometheus** - Установка мониторинга
- **loki** - Установка логирования
- **argocd** - Установка ArgoCD
- **ingress-rules** - Настройка правил ingress

---

## 🚀 Быстрый старт

### Предварительные требования

#### На машине, с которой запускаете Ansible:
- **Linux** (Ubuntu 20.04+, CentOS 8+, или аналогичный)
- **Python 3.6+**
- **pip** (менеджер пакетов Python)
- **git**

#### На целевых серверах:
- **Ubuntu 20.04+** или **CentOS 8+**
- **Доступ по SSH** (ключи или пароль)
- **Пользователь с правами sudo** (или root)
- **Минимум 2 ГБ RAM** на ноду
- **Минимум 20 ГБ свободного места**

### 📋 Пошаговая инструкция

#### 1️⃣ Клонирование репозитория
```bash
git clone <URL_РЕПОЗИТОРИЯ>
cd vs2
```

#### 2️⃣ Установка Ansible и зависимостей
```bash
# Установка Ansible
pip install ansible

# Установка необходимых коллекций
ansible-galaxy collection install -r Ansible/Playbooks/RKE2/collections/requirements.yaml
```

#### 3️⃣ Настройка инвентаря серверов

Отредактируйте файл `Ansible/Playbooks/RKE2/inventory/hosts.ini`:

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

**Пояснение:**
- **servers** - Master-ноды Kubernetes (рекомендуется минимум 3 для высокой доступности)
- **agents** - Worker-ноды для запуска приложений
- **ansible_user** - пользователь для SSH подключения

#### 4️⃣ Настройка переменных

Отредактируйте файл `Ansible/Playbooks/RKE2/inventory/group_vars/all.yaml`:

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

#### 5️⃣ Проверка подключения
```bash
cd Ansible/Playbooks/RKE2
ansible -i inventory/hosts.ini all -m ping
```

**Ожидаемый результат:**
```
server1 | SUCCESS => {"changed": false, "ping": "pong"}
server2 | SUCCESS => {"changed": false, "ping": "pong"}
...
```

#### 6️⃣ Запуск установки
```bash
ansible-playbook -i inventory/hosts.ini site.yaml
```

**Время выполнения:** 15-30 минут в зависимости от количества серверов и скорости интернета.

---

## 🔧 Что происходит во время установки

### Этап 1: Подготовка серверов
- Включение IP forwarding
- Обновление системы
- Установка необходимых пакетов

### Этап 2: Установка RKE2
- Скачивание и установка RKE2
- Настройка конфигурационных файлов
- Запуск первого master-сервера

### Этап 3: Расширение кластера
- Добавление дополнительных master-нод
- Добавление worker-нод
- Настройка высокой доступности

### Этап 4: Установка инфраструктуры
- MetalLB (балансировщик нагрузки)
- Longhorn (система хранения)
- NGINX Ingress Controller

### Этап 5: Установка мониторинга
- Prometheus (сбор метрик)
- Grafana (визуализация)
- Loki (централизованное логирование)

### Этап 6: Установка GitOps
- ArgoCD для управления приложениями

---

## ✅ Проверка работоспособности

### 1. Проверка нод кластера
```bash
# Скопируйте kubeconfig с первого сервера
scp ansible@192.168.1.10:/etc/rancher/rke2/rke2.yaml ~/kubeconfig

# Настройте переменную окружения
export KUBECONFIG=~/kubeconfig

# Проверьте ноды
kubectl get nodes
```

### 2. Проверка подов
```bash
kubectl get pods -A
```

### 3. Проверка сервисов
```bash
kubectl get svc -A
```

---

## 🌐 Доступ к сервисам

После установки будут доступны следующие сервисы:

| Сервис | URL | Логин/Пароль |
|--------|-----|---------------|
| **Grafana** | http://grafana.local | admin / admin123 |
| **Prometheus** | http://prometheus.local | - |
| **ArgoCD** | http://argocd.local | admin / admin123 |

**Примечание:** Добавьте записи в `/etc/hosts` или настройте DNS для доступа по доменным именам.

---

## 🔒 Безопасность

### Рекомендации для production:

1. **Измените пароли по умолчанию:**
   ```yaml
   grafana_admin_password: "СЛОЖНЫЙ_ПАРОЛЬ"
   argocd_admin_password: "СЛОЖНЫЙ_ПАРОЛЬ"
   ```

2. **Настройте SSL/TLS сертификаты**

3. **Ограничьте доступ к API серверу**

4. **Регулярно обновляйте компоненты**

---

## 🛠️ Устранение неполадок

### Частые проблемы:

#### ❌ Ошибка SSH подключения
```bash
# Проверьте SSH доступ
ssh ansible@192.168.1.10

# Проверьте права пользователя
sudo -l
```

#### ❌ Недостаточно места на диске
```bash
# Проверьте свободное место
df -h

# Очистите кэш пакетов
sudo apt clean  # для Ubuntu
sudo yum clean all  # для CentOS
```

#### ❌ Проблемы с сетью
```bash
# Проверьте доступность портов
telnet 192.168.1.10 22
telnet 192.168.1.10 6443
```

#### ❌ Ошибки в Ansible
```bash
# Запустите с подробным выводом
ansible-playbook -i inventory/hosts.ini site.yaml -vvv

# Запустите только определенную роль
ansible-playbook -i inventory/hosts.ini site.yaml --tags prepare-nodes
```

---

## 📚 Полезные команды

### Управление кластером:
```bash
# Проверка статуса нод
kubectl get nodes

# Просмотр подов
kubectl get pods -A

# Просмотр сервисов
kubectl get svc -A

# Просмотр ingress
kubectl get ingress -A
```

### Мониторинг:
```bash
# Просмотр метрик Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# Доступ к Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

### ArgoCD:
```bash
# Доступ к ArgoCD UI
kubectl port-forward -n argocd svc/argocd-server 8080:80

# Получение пароля ArgoCD
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

## 🔄 Обновление кластера

### Обновление RKE2:
```bash
# На всех серверах
sudo systemctl stop rke2-server
sudo systemctl stop rke2-agent

# Скачайте новую версию
curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION=v1.29.4+rke2r1 sh -

# Запустите сервисы
sudo systemctl start rke2-server
sudo systemctl start rke2-agent
```

### Обновление компонентов:
```bash
# Обновите Helm чарты
helm repo update

# Обновите приложения через ArgoCD
# (через веб-интерфейс или CLI)
```

---

## 📞 Поддержка

### Логи и отладка:
```bash
# Логи RKE2 сервера
sudo journalctl -u rke2-server -f

# Логи RKE2 агента
sudo journalctl -u rke2-agent -f

# Логи kubelet
sudo journalctl -u kubelet -f
```

### Полезные файлы:
- `/etc/rancher/rke2/rke2.yaml` - конфигурация кластера
- `/var/lib/rancher/rke2/` - данные кластера
- `/etc/rancher/rke2/config.yaml` - конфигурация RKE2

---

