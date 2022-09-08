**Проблемы при запуске Prometheus Node Exporter**

Если Pods Node Exporter не создаются выполните команду
```
kubectl describe ds node-exporter
```

Вы увидите следующие ошибки
```
Error creating: admission webhook "validation.gatekeeper.sh" denied the request: [psp-host-filesystem] HostPath volume {"hostPath": {"path": "/", "type": ""}, "name": "root"} is not allowed, pod: node-exporter-6rwmd. Allowed path: [{"pathPrefix": "/psp", "readOnly": true}], please follow the article https://vk.cc/cfc8TH to get more information
[psp-host-filesystem] HostPath volume {"hostPath": {"path": "/proc", "type": ""}, "name": "proc"} is not allowed, pod: node-exporter-fcwxm. Allowed path: [{"pathPrefix": "/psp", "readOnly": true}], please follow the article https://vk.cc/cfc8TH to get more information
[psp-host-filesystem] HostPath volume {"hostPath": {"path": "/sys", "type": ""}, "name": "sys"} is not allowed, pod: node-exporter-fcwxm. Allowed path: [{"pathPrefix": "/psp", "readOnly": true}], please follow the article https://vk.cc/cfc8TH to get more information
[psp-host-namespace] Sharing the host namespace is not allowed: node-exporter-fcwxm, please follow the article https://vk.cc/cfc8TH to get more information
```

Проблемы связана с работой GateKeeper в последних версиях кластеров K8s в облаке VK

Просмотреть правила GateKeeper можно командой
```
kubectl get constrainttemplate
```

Удалите существующие правила
```
kubectl delete constrainttemplate k8spsphostfilesystem
kubectl delete constrainttemplate k8spsphostnamespace
```

Через несколько минут конфигурация GateKeeper обновится и Pods Node Exporter будут созданы.
