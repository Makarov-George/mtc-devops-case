
# PROGRESS — журнал работы над кейсом

> Этот файл обновляется по мере работы. Служит для быстрого
> восстановления контекста при возврате к проекту.

## ✅ Что уже сделано

### Окружение
- Windows 10 + Docker Desktop (WSL2)
- Установлены: kubectl v1.36.1, kind v0.23.0, helm v3.15.0
- kind-кластер `mtc-cluster` (v1.30.0) запущен и работает
- StorageClass `standard` (rancher.io/local-path) доступен

### Веб-приложение
- Namespace `demo-app` создан
- nginx 1.27-alpine, 2 реплики, статус Running
- ConfigMap с кастомным nginx.conf:
  - `/` → `Hello World!`
  - `/healthz` → `ok`
  - `/nginx_status` → stub_status для Prometheus
  - access-логи → stdout
- Service `web-app` (ClusterIP, port 80)
- Проверено: `kubectl port-forward ... 8080:80` + `curl http://localhost:8080` → `Hello World!`

### Gateway API
- Envoy Gateway v1.1.0 установлен (namespace `envoy-gateway-system`)
- GatewayClass `eg` → Accepted=True
- EnvoyProxy `nodeport-config` (в namespace `demo-app`, тип NodePort, порт 30000)
- Gateway `eg` (namespace `demo-app`) → PROGRAMMED=True, address=172.18.0.2
- HTTPRoute `web-app` (namespace `demo-app`) → `/` → service `web-app:80`
- **Проверено: `curl.exe http://localhost:30000` → `Hello World!`** ✅

### Git
- Локальный репозиторий инициализирован
- GitHub: https://github.com/<username>/mtc-devops-case (публичный)
- Все текущие манифесты и README запушены
- README содержит Mermaid-схему архитектуры

## 🚧 Что НЕ сделано (TODO)

### Обязательная часть
- [ ] **Мониторинг (Prometheus)** — не начат
  - Планируется: kube-prometheus-stack (или Prometheus+Grafana отдельно)
  - ServiceMonitor для nginx-exporter
  - Проверка метрик через PromQL
- [ ] **Логирование (Fluent Bit + Loki)** — не начат
  - Fluent Bit DaemonSet, сбор stdout-логов nginx
  - Loki как хранилище
  - Проверка через Grafana
- [ ] **Автоматизация** — не начата
  - `deploy.sh` — одна команда для развёртывания
  - `destroy.sh` — одна команда для удаления
  - `Makefile` — `make deploy/destroy/test`
- [ ] **README** — дописать разделы про мониторинг, логирование, автоматизацию
- [ ] **Паспорт проекта** — PDF/DOCX ≤4 стр., не начат
- [ ] **Архив для сдачи** — ZIP с `Ссылка.txt` и `Паспорт.pdf`

### Бонусы (по желанию)
- [ ] Grafana dashboards
- [ ] CI/CD через GitHub Actions
- [ ] TLS через cert-manager
- [ ] Traffic splitting (2 версии nginx)
- [ ] Метрики Envoy через ServiceMonitor

## 🔧 Рабочие команды (проверены)

```powershell
# Проверить кластер
kubectl get nodes
kubectl get pods -n demo-app
kubectl get gateway -n demo-app
kubectl get httproute -n demo-app

# Проверить приложение через Gateway
curl.exe http://localhost:30000
curl.exe http://localhost:30000/healthz

# Проверить приложение напрямую (port-forward)
kubectl port-forward -n demo-app svc/web-app 8080:80
# в другом окне:
curl.exe http://localhost:8080
```

## 📁 Структура репозитория

```text
D:\MTS_Hack\  (корень репо)
├── README.md
├── PROGRESS.md                ← этот файл
├── kind-config.yaml           ← конфиг kind-кластера
├── k8s/
│   ├── app/
│   │   ├── namespace.yaml
│   │   ├── configmap-nginx.yaml
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── gateway/
│       ├── gatewayclass.yaml
│       ├── envoyproxy-nodeport.yaml
│       ├── gateway.yaml
│       └── httproute.yaml
├── monitoring/                ← пусто (Prometheus значения будут тут)
├── scripts/                   ← пусто (deploy.sh, destroy.sh)
└── docs/                      ← пусто (паспорт, схема)
```
## 🎯 Следующий шаг

Шаг 4: Prometheus.
Решено: устанавливаем kube-prometheus-stack (Helm) или
Prometheus+Grafana отдельно. Первым делом:

1. helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

2. helm install kps prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

## ⚠️ Известные проблемы / грабли
 1. GatewayClass eg не создался автоматически при установке Envoy Gateway v1.1.0.
    Решение: создать вручную (k8s/gateway/gatewayclass.yaml).

 2. EnvoyProxy в другом namespace не работает — parametersRef не поддерживает namespace.
    Решение: EnvoyProxy создан в том же namespace demo-app, что и Gateway.

 3. kind не имеет cloud-provider → LoadBalancer Service зависает.
    Решение: EnvoyProxy с типом NodePort + порт 30000 (проброшен в kind-config.yaml).

 4. curl в PowerShell — это алиас Invoke-WebRequest, вывод «красивый».
    Для чистого ответа: curl.exe.

 5. kubectl apply -f k8s/app/ применяет файлы в алфавитном порядке,
    namespace может создаться позже других ресурсов → ошибка.
    Решение: применять повторно или через kustomization.yaml.

Последнее обновление: 01.10.2026, вечер.