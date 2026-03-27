Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки»  
Студент: Кочнев Михаил Сергеевич  
Задание 1: Настройка балансировки Round-robin на 4 уровне (TCP)  
Описание:  
Настроена балансировка трафика между двумя серверами Python (порты 8081 и 8082) на уровне L4 с использованием алгоритма Round-robin.  
Конфигурация HAProxy (/etc/haproxy/haproxy.cfg): 
```haproxy  
listen python_l4  
    bind :80  
    mode tcp  
    balance roundrobin  
    server s1 127.0.0.1:8081 check  
    server s2 127.0.0.1:8082 check  
```

![Задание1](https://github.com/user-attachments/assets/b5db360a-a59b-45a6-abe6-588ba55a647b)

![Задание1](https://github.com/user-attachments/assets/006bbd9c-8389-46cc-adf5-401b62fb7329)
    
Скриншот работы (чередование запросов):  

<img width="415" height="334" alt="Снимок экрана 2026-03-27 в 18 56 59" src="https://github.com/user-attachments/assets/f19aa5df-8c46-46ed-89f9-c83f1896910b" />

<img width="421" height="342" alt="Снимок экрана 2026-03-27 в 18 56 41" src="https://github.com/user-attachments/assets/b3bf45bb-fbb0-4fb7-95a2-8c2332dbe9a9" />  

<img width="411" height="326" alt="Снимок экрана 2026-03-27 в 18 56 03" src="https://github.com/user-attachments/assets/bc433b2d-3a7e-4234-a5f0-4ad2a472c0a7" />


Задание 2: Настройка Weighted Round Robin на 7 уровне (HTTP) с ACL  
Описание:  
Настроена балансировка для трех серверов с весами (2, 3, 4). Доступ разрешен только при обращении к домену example.local. Запросы к другим доменам (например, localhost) отклоняются с ошибкой 503.  
Настройка локального домена (/etc/hosts):  
127.0.0.1 localhost  
127.0.0.1 example.local  
Итоговая конфигурация HAProxy:

```haproxy  
frontend http_front  
    bind :80  
    mode http  
    acl is_example_local hdr(host) -i example.local  
    use_backend python_weighted if is_example_local  
backend python_weighted  
    mode http  
    balance roundrobin  
    server s1 127.0.0.1:8081 weight 2 check  
    server s2 127.0.0.1:8082 weight 3 check  
    server s3 127.0.0.1:8083 weight 4 check  
```
![Задание 2]<img width="477" height="262" alt="Снимок экрана 2026-03-26 в 20 37 22" src="https://github.com/user-attachments/assets/00a30f15-adbb-47c5-bb9d-1a30f84c5051" />    


<img width="477" height="340" alt="Снимок экрана 2026-03-26 в 20 37 44" src="https://github.com/user-attachments/assets/e8fb6aca-717e-4d1c-a4e7-5d52f4d1d311" />    

Проверка работы ACL (200 OK и 503 Error):  
На скриншоте ниже видно, что при обращении к example.local сервер отвечает успешно, а при обращении к localhost возвращается ошибка 503.  

<img width="477" height="206" alt="Снимок экрана 2026-03-26 в 20 36 31" src="https://github.com/user-attachments/assets/c3cac1ef-cb41-4e76-827f-33b8ff647ddc" />  


Проверка распределения весов:  
bash  
# Проверка доступности домена
curl -I http://example.local

# Цикл для проверки весов (9 запросов)
for i in {1..9}; do curl -s http://example.local > /dev/null; done

После выполнения цикла из 9 запросов (for i in {1..9}; do curl -s http://example.local > /dev/null; done), логи Python-серверов  подтверждают распределение:    
<img width="410" height="336" alt="Снимок экрана 2026-03-27 в 19 12 08" src="https://github.com/user-attachments/assets/0496d1cf-b8b8-473d-9b4c-46e5facda085" />


Сервер s1 (8081): 2 запроса  
<img width="412" height="328" alt="Снимок экрана 2026-03-27 в 19 12 24" src="https://github.com/user-attachments/assets/cf48933a-02ad-4eac-879c-48be10720065" />


Сервер s2 (8082): 3 запроса  

<img width="417" height="334" alt="Снимок экрана 2026-03-27 в 19 12 42" src="https://github.com/user-attachments/assets/7a3e7c09-127c-484d-9a57-983138ddbb8e" />

Сервер s3 (8083): 4 запроса  
<img width="419" height="331" alt="Снимок экрана 2026-03-27 в 19 12 59" src="https://github.com/user-attachments/assets/aa51ea80-85ac-405b-baed-00340c3da46f" />



