# HPA

Создаем деплоймент с тестовым приложением

```bash
kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=100m --expose --port=80
```

Создаем автоскейлер

```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=5
```

Смотрим на текущее количество подов

```bash
kubectl get pod
```

Видим один под
```bash
NAME                          READY   STATUS    RESTARTS   AGE
php-apache-566d7644df-z9dtt   1/1     Running   0          15s
```

Смотрим на HPA

```bash
kubectl get hpa
```

Видим созданную HPA
```bash
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   1%/50%    1         5         1          32s
```

Она будет скейлить поды, как только их использование цпу начнет составлять 50% от реквестов

Создаем нагрузку

```bash
kubectl run load-generator --image=busybox -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"
```

Проверяем текущее потребление подом цпу

```
kubectl top pod
```

Видим что она подросла
```bash
NAME                          CPU(cores)   MEMORY(bytes)
php-apache-566d7644df-z9dtt   936m         11Mi
```

Ждем когда начнет работать автоскейл

```bash
kubectl get pod -w
```

Через какое то время видим, что подов стало 5

```bash
NAME                              READY   STATUS    RESTARTS   AGE
load-generator-6b9cf94758-5qmbx   1/1     Running   0          2m16s
php-apache-566d7644df-4zvv7       1/1     Running   0          108s
php-apache-566d7644df-kv662       1/1     Running   0          93s
php-apache-566d7644df-tg8qw       1/1     Running   0          108s
php-apache-566d7644df-z9dtt       1/1     Running   0          13m
php-apache-566d7644df-zlwd7       1/1     Running   0          108s
```
Отлично, автоскейл сработал

Удираем нагрузку

```bash
kubectl delete deployment load-generator
```

Проверяем нагрузку на поды

```bash
kubectl top pod
```

Через какое то время замечаем что она упала

```bash
NAME                          CPU(cores)   MEMORY(bytes)
php-apache-566d7644df-4zvv7   1m           11Mi
php-apache-566d7644df-kv662   1m           11Mi
php-apache-566d7644df-tg8qw   1m           11Mi
php-apache-566d7644df-z9dtt   1m           11Mi
php-apache-566d7644df-zlwd7   1m           11Mi
```

Ну и смотрим как автоскейлер отработает в обратную сторону

```bash
kubectl get pod -w
```

Видим что ненужные поды умирают (в течении 5 минут)

```bash
NAME                          READY   STATUS        RESTARTS   AGE
php-apache-566d7644df-4zvv7   0/1     Terminating   0          8m59s
php-apache-566d7644df-kv662   0/1     Terminating   0          8m44s
php-apache-566d7644df-tg8qw   0/1     Terminating   0          8m59s
php-apache-566d7644df-z9dtt   1/1     Running       0          20m
php-apache-566d7644df-zlwd7   0/1     Terminating   0          8m59s
```

Автоскейлер вернул все к первоначальному варианту с одним подом
