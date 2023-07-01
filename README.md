# k8s2
Kubernetes 2

**Архитектура k8s**

**Компоненты:**
- kube-api-server
- kube-controller-manager
- kube-scheduler
- etcd
- kube-proxy
- kubelet

**kube-api-server**

**Сервер API** — компонент панели управления Kubernetes, который представляет API Kubernetes

**API-сервер** — это клиентская часть панели управления Kubernetes. Основной реализацией API-сервера Kubernetes является kube-apiserver. kube-apiserver предназначен для горизонтального масштабирования — развёртывание на несколько экземпляров. Вы можете запустить несколько экземпляров kube-apiserver и сбалансировать трафик между этими экземплярами

**etcd** — распределённое и высоконадёжное хранилище данных в формате «ключ-значение», которое используется, как основное хранилище всех данных кластера в Kubernetes

**kube-controller-manager** — компонент k8s, который запускает процессы контроллера:
- контроллер узла (Node Controller): уведомляет и реагирует на сбои узла
- контроллер репликации (Replication Controller): поддерживает правильное количество подов для каждого объекта контроллера репликации в системе
- контроллер конечных точек (Endpoints Controller): заполняет объект конечных точек (Endpoints), то есть связывает сервисы (Services) и поды (Pods)
- контроллеры учётных записей и токенов (Account & Token Controllers): создают стандартные учётные записи и токены доступа API для новых пространств имён 

**kube-scheduler** — компонент k8s, который отслеживает созданные поды без привязанного узла и выбирает узел, на котором они должны работать При планировании развёртывания подов на узлах, учитывается множество факторов, включая требования к ресурсам, ограничения, связанные с аппаратными или программными политиками, принадлежности (affinity) и непринадлежности (anti-affinity) узлов или подов, местонахождения данных, предельных сроков

**kube-proxy** — сетевой прокси, работающий на каждом узле в кластере и реализующий часть концепции сервис kube-proxy конфигурирует правила сети на узлах. При помощи них разрешаются сетевые подключения к вашими подам изнутри и снаружи кластера kube-proxy использует уровень фильтрации пакетов в операционной системе, если он доступен. В другом случае kube-proxy сам обрабатывает передачу сетевого трафика

**kubelet** — агент, работающий на каждом узле в кластера. Он следит за тем, чтобы контейнеры были запущены в поде Утилита kubelet принимает набор PodSpecs и гарантирует работоспособность и исправность определённых в них контейнеров. Агент kubelet не отвечает за контейнеры, не созданные Kubernetes

**Сетевые плагины**

Для реализации сетевого взаимодействия используются сетевые плагины — CNI

**CNI (Container Network interface)** представляет собой спецификацию для организации универсального сетевого решения для Linux- контейнеров. Он включает в себя плагины, отвечающие за различные функции при настройке сети пода. Плагин CNI — это исполняемый файл, соответствующий спецификации

Одной из главных ценностей CNI, конечно, являются сторонние плагины, обеспечивающие поддержку различных современных решений для Linux-контейнеров. Среди них:
- Project Calico
- Weave
- Flannel

**Установка kubeadm, kubelet, kubectl**

Устанавливаем зависимости для apt:
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```
Настраиваем репозиторий:
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg
https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg]
https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee
/etc/apt/sources.list.d/kubernetes.list
```
Устанавливаем пакеты:
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

***Создание кластера***

Инициализируем первую мастер ноду:
```
kubeadm init
```
Если всё прошло успешно, то в консоль выведется команда для присоединения нод к кластеру, выполняем её:
```
kubeadm join <control-plane-host>:<control-plane-port> --token <token>
--discovery-token-ca-cert-hash sha256:<hash>
```
Устанавливаем CNI, например weave:
```
kubectl apply -f
https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

**Helm** — пакетный менеджер k8s. Позволяет объединять несколько yaml-файлов и шаблонизировать их с помощью встроенных функций и параметров. Такой пакет называется Chart. Имеет функционал отката релиза, тестирования, хранит текущее состояние релиза внутри кластера

Установка Helm производится с помощью скрипта установки:
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```
Helm можно использовать для установки чартов из публичных репозиториев:
```
helm repo add stable https://charts.helm.sh/stable
helm search repo wordpress
helm install my-wordpress stable/wordpress
```
И для создания, и деплоя своих собственных чартов:
```
helm create mychart
```
Структура в директории с чартом должна иметь вид:
```
mychart/
  Chart.yaml
  values.yaml
  templates/
```
- Chart.yaml содержит метаданные о самом чарте — название, версия, автор и т. д.
- values.yaml — значения для установки по умолчанию
- templates/*.yaml — те объекты, которые будут загружены при установке чарта

**Chart.yaml**
- apiVersion — указывает версию API для helm
- name — название чарта, именно оно будет фигурировать в поиске при загрузке чарта в репозиторий
- version — версия чарта
- appVersion — версия приложения внутри чарта, они не всегда могут совпадать
```
apiVersion: v2
name: frontend-chart
description: A Helm chart for
Kubernetes
type: application
version: 0.1.0
appVersion: 0.1.0
```
**values.yaml**
- В этом файле хранятся любые параметры, которые потребуется переопределять в разных инсталляциях
- Хорошим тоном в публичных чартах считается иметь работоспособные параметры по умолчанию
```
replicas: 2
database:
host: localhost
port: 5432
users:
- root
- test
- web
```
**пример шаблона yaml-файла**
- Создадим в папке templates configmap.yaml с подобным содержанием
- Helm имеет встроенные объекты, например, Release.Name. Полный список есть в официальной документации
- Использование параметров из values.yaml возможно с помощью {{ .Values. }}
```
apiVersion: v1
kind: ConfigMap
metadata:
name: {{ .Release.Name }}-configmap
data:
myvalue: "Hello World"
test: {{ .Values.replicas | quote }}
```
**Helm: деплой проекта**

Установим наш чарт
```
helm install mycahart ./mychart
```
Проверим его манифест
```
helm get manifest mycahart
```
Удалим чарт
```
helm uninstall mycahart
```





