# SAGA
Домашнее задание. Распределенные транзакции

**Задание**

Можно использовать приведенный ниже сценарий для интернет-магазина или придумать свой.
Дефолтный сценарий:
Реализовать сервисы "Платеж", "Склад", "Доставка".
Для сервиса "Заказ", в рамках метода "создание заказа" реализовать механизм распределенной транзакции (на основе Саги или двухфазного коммита).
Во время создания заказа необходимо:
- в сервисе "Платеж" убедиться, что платеж прошел
- в сервисе "Склад" зарезервировать конкретный товар на складе
- в сервисе "Доставка" зарезервировать курьера на конкретный слот времени.
Если хотя бы один из пунктов не получилось сделать, необходимо откатить все остальные изменения.
На выходе должно быть:
- Описание того, какой паттерн для реализации распределенной транзакции использовался.
- Команда установки приложения (из helm-а или из манифестов).Обязательно указать в каком namespace нужно устанавливать и команду создания namespace, если это важно для сервиса.
- Тесты в postman.В тестах обязательно использование домена arch.homework в качестве initial значения {{baseUrl}}.

**Решение**
В качестве схемы реализации был применен паттерн Сага с хареографией, где сервисы буликуют в шину сообщений события типа: Заказ оплачен, оплата заказа не прошла, товар зарезервирован, резерв товара отменен, слот доставки зарезервирован и слот доставки отменен.
![image](https://user-images.githubusercontent.com/60660331/188972991-1fb281a1-ad22-4cf5-bd0c-b02d981415ef.png)

В качестве брокера сообщений был использован RabbitMQ. 
В качестве СУБД сервисов был использован PostgreSQL.
В качестве Gateway используется Ingress NGINX.

Инструкция по установке:
1. Установить NGINX Ingress
```
helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace --set controller.admissionWebhooks.enabled=false
```
2. Скопировать репозиторий командой Git
```
git clone https://github.com/Flipmachin1001/SAGA.git
```
3. Перейти в созданный каталог
```
cd SAGA
```
4. Установить сервисы через Helm:
```
helm install saga ./helm-chart
```
5. Дождаться пока установиться RabbitMQ. Для проверки готовности выполнить команду:
```
kubectl get pod/saga-rabbitmq-0
```
6. Запустить тесты Postman командой:
```
newman run SAGA.postman_collection.json --delay-request 500
```
7. Удалить сервисы через Helm:
```
helm uninstall saga
```
