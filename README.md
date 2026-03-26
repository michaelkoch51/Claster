Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки»  
Студент: Кочнев Михаил Сергеевич  
Задание 1: Настройка балансировки Round-robin на 4 уровне (TCP)  
Описание:  
Настроена балансировка трафика между двумя серверами Python (порты 8081 и 8082) на уровне L4 с использованием алгоритма Round-robin.  
Конфигурация HAProxy (/etc/haproxy/haproxy.cfg):  
listen python_l4  
    bind :80  
    mode tcp  
    balance roundrobin  
    server s1 127.0.0.1:8081 check  
    server s2 127.0.0.1:8082 check  

<img width="471" height="225" alt="Снимок экрана 2026-03-26 в 20 20 27" src="https://github.com/user-attachments/assets/b5db360a-a59b-45a6-abe6-588ba55a647b" />  

<img width="470" height="371" alt="Снимок экрана 2026-03-26 в 20 20 34" src="https://github.com/user-attachments/assets/006bbd9c-8389-46cc-adf5-401b62fb7329" />  
    
Скриншот работы (чередование запросов):  

<img width="467" height="374" alt="Снимок экрана 2026-03-26 в 20 19 53" src="https://github.com/user-attachments/assets/7682aed6-d567-4914-a2ab-f01eef634103" />


Задание 2: Настройка Weighted Round Robin на 7 уровне (HTTP) с ACL  
Описание:  
Настроена балансировка для трех серверов с весами (2, 3, 4). Доступ разрешен только при обращении к домену example.local. Запросы к другим доменам (например, localhost) отклоняются с ошибкой 503.  
Настройка локального домена (/etc/hosts):  
127.0.0.1 localhost  
127.0.0.1 example.local  
Итоговая конфигурация HAProxy:  
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

<img width="477" height="262" alt="Снимок экрана 2026-03-26 в 20 37 22" src="https://github.com/user-attachments/assets/00a30f15-adbb-47c5-bb9d-1a30f84c5051" />    


<img width="477" height="340" alt="Снимок экрана 2026-03-26 в 20 37 44" src="https://github.com/user-attachments/assets/e8fb6aca-717e-4d1c-a4e7-5d52f4d1d311" />    

Проверка работы ACL (200 OK и 503 Error):  
На скриншоте ниже видно, что при обращении к example.local сервер отвечает успешно, а при обращении к localhost возвращается ошибка 503.  

<img width="477" height="206" alt="Снимок экрана 2026-03-26 в 20 36 31" src="https://github.com/user-attachments/assets/c3cac1ef-cb41-4e76-827f-33b8ff647ddc" />  


Проверка распределения весов:  
После выполнения цикла из 9 запросов (for i in {1..9}; do curl -s http://example.local > /dev/null; done), логи Python-серверов  подтверждают распределение:  
Сервер s1 (8081): 2 запроса  
Сервер s2 (8082): 3 запроса  
Сервер s3 (8083): 4 запроса  


<img width="484" height="99" alt="Снимок экрана 2026-03-26 в 20 49 46" src="https://github.com/user-attachments/assets/82f517e3-981d-4d05-bc05-5be54e3a0740" />  

<img width="487" height="239" alt="Снимок экрана 2026-03-26 в 20 50 11" src="https://github.com/user-attachments/assets/71aae7d6-11ee-4d7f-9e60-e7f7bc68af3b" />  

<img width="357" height="123" alt="Снимок экрана 2026-03-26 в 20 49 38" src="https://github.com/user-attachments/assets/f617f1d8-4f31-4fd2-b562-83fc262e48ef" />  
