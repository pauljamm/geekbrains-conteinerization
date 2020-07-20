# Инструкция для лекции 8 - CI/CD

## Подготовка

* Создаем нэймспэйс для раннера

```bash
kubectl create ns gitlab
```

* Меняем регистрационный токен
Для этого открываем gitlab-runner/gitlab-runner.yaml
Там ищем <CHANGE ME> и вставляем вместо него токен,
который мы взяли в настройках проекта на Gitlab

* Применяем манифесты для раннера

```bash
kubectl apply --namespace gitlab -f gitlab-runner/gitlab-runner.yaml
```

* Создаем нэймспэйсы для приложения

```bash
kubectl create ns stage
kubectl create ns prod
```

* Создаем авторизационные объекты, чтобы раннер мог деплоить в наши нэймспэйсы

```bash
kubectl create sa deploy --namespace stage
kubectl create rolebinding deploy --serviceaccount stage:deploy --clusterrole edit --namespace stage
kubectl create sa deploy --namespace prod
kubectl create rolebinding deploy --serviceaccount prod:deploy --clusterrole edit --namespace prod
```

* Получаем токены для деплоя в нэймспэйсы

```bash
export NAMESPACE=stage; kubectl get secret $(kubectl get sa deploy --namespace $NAMESPACE -o jsonpath='{.secrets[0].name}') --namespace $NAMESPACE -o jsonpath='{.data.token}
export NAMESPACE=prod; kubectl get secret $(kubectl get sa deploy --namespace $NAMESPACE -o jsonpath='{.secrets[0].name}') --namespace $NAMESPACE -o jsonpath='{.data.token}
```

Из этих токенов нужно создать переменные в проекте в Gitlab с именами
K8S_STAGE_CI_TOKEN и K8S_PROD_CI_TOKEN соответственно.

* Создаем секреты для авторизации Kubernetes в Gitlab registry

```bash
kubectl create secret docker-registry gitlab-registry --docker-server=registry.gitlab.com --docker-username=<USERNAME> --docker-password=<PASSWORD> --docker-email=admin@admin.admin --namespace stage
kubectl create secret docker-registry gitlab-registry --docker-server=registry.gitlab.com --docker-username=<USERNAME> --docker-password=<PASSWORD> --docker-email=admin@admin.admin --namespace prod
```

* Патчим дефолтный сервис аккаунт для автоматического использование pull secretа

```bash
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "gitlab-registry"}]}' -n stage
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "gitlab-registry"}]}' -n prod
```

## Запуск приложения

* Создаем манифесты для БД в стейдже и проде

```bash
kubectl apply --namespace stage -f app/kube/postgres/
kubectl apply --namespace prod -f app/kube/postgres/
```

* Меняем хост в ингрессе приложения и применяем манифесты
Для этого открываем app/kube/ingress.yaml
Там ищем <CHANGE ME> и вставляем вместо него stage

Далее применяем на стэйдж

```bash
kubectl apply --namespace stage -f app/kube
```

Повторяем для прода. Открываем тот же файл ingress и
вставляем вместо stage prod

Далее применяем на prod

```bash
kubectl apply --namespace prod -f app/kube
```

