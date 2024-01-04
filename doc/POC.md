Argo CD - це інструмент управління і розгортанням (GitOps) для Kubernetes, призначений для автоматизації процесу розгортання та оновлення додатків у кластері Kubernetes. GitOps - це методологія, де конфігурація системи та стан додатків зберігається у репозиторії Git, а система автоматично використовує цей репозиторій для синхронізації стану кластера з бажаним станом.

    1. Підготуємо в окремий локальний кластер для встановлення ArgoCD. Налаштуємо його:

```bash 
$k3d cluster create argo
INFO[0000] ...                                               
kubectl cluster-info
kubectl cluster-info
Kubernetes control plane is running at https://0.0.0.0:44619
CoreDNS is running at https://0.0.0.0:44619/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://0.0.0.0:44619/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

    2. Інсталяція
ArgoCD можна встановити за допомогою `Helm`, або використаємо скрипт з офіційного репозиторію [ArgoCD](https://argo-cd.readthedocs.io/en/stable/#quick-start), ми використаємо другий варіант. Спочатку створимо неймспейс в якому буде встановлено систему, потім скористаємось скриптом (маніфестом) для інсталяції та перевіримо стан системи після встановлення:

```bash
    $kubectl create namespace argocd
namespace/argocd created
    $kubectl get ns
NAME              STATUS   AGE
default           Active   10m
kube-system       Active   10m
kube-public       Active   10m
kube-node-lease   Active   10m
argocd            Active   25s

    $kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

    $kubectl get all -n argocd
NAME                                                   READY   STATUS    RESTARTS   AGE
pod/argocd-redis-b5d6bf5f5-gbbjf                       1/1     Running   0          57s
pod/argocd-notifications-controller-db4f975f8-p45b5    1/1     Running   0          57s
pod/argocd-applicationset-controller-dc5c4c965-prn2g   1/1     Running   0          57s
pod/argocd-repo-server-579cdc7849-b9lg5                1/1     Running   0          57s
pod/argocd-dex-server-9769d6499-fjlb2                  1/1     Running   0          57s
pod/argocd-application-controller-0                    1/1     Running   0          56s
pod/argocd-server-557c4c6dff-lmw4s                     1/1     Running   0          56s
```

### перевіримо статус контейнерів

```bash
    $kubectl get pod -n argocd -w
NAME                                               READY   STATUS    RESTARTS   AGE
argocd-redis-b5d6bf5f5-gbbjf                       1/1     Running   0          2m43s
argocd-notifications-controller-db4f975f8-p45b5    1/1     Running   0          2m43s
argocd-applicationset-controller-dc5c4c965-prn2g   1/1     Running   0          2m43s
argocd-repo-server-579cdc7849-b9lg5                1/1     Running   0          2m43s
argocd-dex-server-9769d6499-fjlb2                  1/1     Running   0          2m43s
argocd-application-controller-0                    1/1     Running   0          2m42s
argocd-server-557c4c6dff-lmw4s                     1/1     Running   0          2m42s
```
    3. Отримаємо доступ до інтерфейсу ArgoCD GUI 
[Отримати доступ](https://argo-cd.readthedocs.io/en/stable/getting_started/#3-access-the-argo-cd-api-server) можна наступними шляхами:  
- Service Type Load Balancer  
- Ingress  
- Port Forwarding 

Скористаємось `Port Forwarding` за допомогою локального порта 8080. В команді ми посилаємось на сервіс `svc/argocd-server` який знаходиться в namespace `-n argocd`. Kubectl автоматично знайде endpoint сервісу та встановить переадресацію портів з локального порту 8080 на віддалений 443 

```bash
    $kubectl port-forward svc/argocd-server -n argocd 8080:443&
[1] 39748
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
Handling connection for 8080
```
ArgoCD за замовчуванням працює з https тому при спробі відкрити [127.0.0.1:8080](https://127.0.0.1:8080/) ми отримаємо помилку сертифікату. В продуктивній системі потрібно встановлювати сертифікати та налаштовувати ці моменти. 

    4. Отримання паролю
Використаємо команду для отримання паролю, вкажемо файл сікрету `argocd-initial-admin-secret` а також формат  виводу `jsonpath="{.data.password}"`, після чого використаємо команду `base64 -d` для повернення паролю в простий текст. Отриманий пароль та логін `admin` вводимо в Web-інтерфейс ArgoCD

```bash
    $kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"|base64 -d;echo
<password>
```
    5. Створимо додаток за допомогою графічного інтерфейсу ArgoCD, після чого додатки будуть автоматично встановлюватись на оновлятись в Kubernetes
- Натискаємо `+ NEW APP` 
- Вводимо ім'я додатку `demo`
- Проект до якого належить додаток оберемо `за замовчуванням`
- Тип синхронізації `Manual` 
- У розділі `SOURCE` тип джерела залишаємо за замовчуванням `GIT`
- Введемо `url` репозиторію, який містить маніфести для розгортання https://github.com/STsariuk/go-demo-app.git (це буде helm charts, або пакет маніфестів який являє собою групу об'єктів для Kubernetes та нашого додатку)
- У полі `Path` введемо шлях до каталогу `helm`   
- В розділі `DESTINATION` вкажемо `url` локального кластеру та `Namespace` demo після чого ArgoCD автоматично визначить параметри додатку використавши маніфести, які знаходяться в репозиторії. В разі бажання змінити значення вручну можна змінити іх значення в розділі `PARAMETERS`.  
- У розділі політика синхронізація вказується як додаток буде синхронізуватись з репозиторієм. Тут необхідно вказати ArgoCD щоб створив новий `Namespace` так як в `helm` цю функцію за замовчуванням прибрали. Ставимо галочку навпроти `AUTO-CREATE NAMESPACE`   
- Створюємо додаток кнопкою `CREATE`

    6.Коротке демо вище вказаних кроків


    7.Прослідкуємо за реакцією ArgoCD на зміни в репозиторії.
 
```bash
$kubectl get svc -n demo
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                 AGE
demo-nats          ClusterIP   None            <none>        4222/TCP,6222/TCP,8222/TCP,7777/TCP,7422/TCP,7522/TCP   8m38s
demo-api           ClusterIP   10.43.187.222   <none>        80/TCP                                                  8m38s
demo-front         ClusterIP   10.43.167.225   <none>        80/TCP                                                  8m38s
ambassador-admin   ClusterIP   10.43.226.22    <none>        8877/TCP                                                8m38s
demo-ascii         ClusterIP   10.43.231.121   <none>        80/TCP                                                  8m38s
demo-img           ClusterIP   10.43.26.76     <none>        80/TCP                                                  8m38s
demo-data          ClusterIP   10.43.133.179   <none>        80/TCP                                                  8m38s
db                 ClusterIP   10.43.125.206   <none>        3306/TCP                                                8m38s
cache              ClusterIP   10.43.55.108    <none>        6379/TCP                                                8m38s
ambassador         NodePort    10.43.123.136   <none>        80:32127/TCP                                            8m38s
```

- Змінимо в файлі репозиторію https://github.com/STsariuk/go-demo-app/blob/main/helm/values.yaml тип шлюзу з `NodePort` на `LoadBalancer` (останній рядок файлу)

```bash
$kubectl get svc -n demo
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                 AGE
demo-nats          ClusterIP      None            <none>        4222/TCP,6222/TCP,8222/TCP,7777/TCP,7422/TCP,7522/TCP   10m
demo-api           ClusterIP      10.43.187.222   <none>        80/TCP                                                  10m
demo-front         ClusterIP      10.43.167.225   <none>        80/TCP                                                  10m
ambassador-admin   ClusterIP      10.43.226.22    <none>        8877/TCP                                                10m
demo-ascii         ClusterIP      10.43.231.121   <none>        80/TCP                                                  10m
demo-img           ClusterIP      10.43.26.76     <none>        80/TCP                                                  10m
demo-data          ClusterIP      10.43.133.179   <none>        80/TCP                                                  10m
db                 ClusterIP      10.43.125.206   <none>        3306/TCP                                                10m
cache              ClusterIP      10.43.55.108    <none>        6379/TCP                                                10m
ambassador         LoadBalancer   10.43.123.136   <pending>     80:32127/TCP                                            10m
```




