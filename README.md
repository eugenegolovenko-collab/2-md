# Домашнее задание к занятию 2 «`Кластеризация и балансировка нагрузки`» - `Евгений Головенко`

---

### Задание 1

- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy, воспользуйтесь материалами к лекции по [ссылке](2/)
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

### Решение 1

`haproxy.cfg`

```
global
        log /dev/log    local0                                                                       
        log /dev/log    local1 notice
        chroot /var/lib/haproxy                                                                      
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s                                                                            
        user haproxy
        group haproxy                                                                                
        daemon
                                                                                                     
        # Default SSL material locations
        ca-base /etc/ssl/certs                                                                       
        crt-base /etc/ssl/private
                                                                                                     
        # See: https://ssl-config.mozilla.org/
        #server=haproxy&server-version=2.0.3&config=intermedia>
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECD>
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POL>
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets                                  

defaults                                                                                             
        log     global
        mode    http                                                                                 
        #option httplog
        option  dontlognull
        timeout connect 5s
        timeout client  50s
        timeout server  50s
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

listen stats  #
        bind *:888
        mode http
        option httplog
         stats enable                                                                                 
        stats uri /stats
        stats refresh 5s                                                                              
        stats realm Haproxy\ Statistics
        
frontend example                                                                        
        mode http
        bind :8088                                                                                    
        option httplog
        default_backend web_servers                                                                   
        #acl ACL_example.com hdr(host) -i example.com
        #use_backend web_servers if ACL_example.com                                                   

backend web_servers    
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 check
        server s2 127.0.0.1:9999 check

listen web_tcp
        mode tcp
        bind *:1325
        balance roundrobin
        option tcplog
        server s1 127.0.0.1:8888 check inter 3s
        server s2 127.0.0.1:9999 check inter 3s
```

`Cкриншот, где видно перенаправление запросов на разные серверы при обращении к HAProxy:`

<img width="587" height="207" alt="Capture1" src="https://github.com/user-attachments/assets/fe40a0ce-69c5-4457-952f-d486873bf2c6" />

<img width="1081" height="705" alt="Capture2" src="https://github.com/user-attachments/assets/f599fbad-1c9d-4639-9268-d6b1feb6bef0" />

`Выключен сервер 1 (IP 192.168.1.227:8888)`

<img width="653" height="256" alt="Capture3" src="https://github.com/user-attachments/assets/271272e0-7a8d-45ff-8b47-9adf03b5efac" />

<img width="1083" height="712" alt="Capture4" src="https://github.com/user-attachments/assets/6690f341-a0e8-4f5a-81d7-0579bd469d7d" />

---

### Задание 2

- Запустите три simple python сервера на своей виртуальной машине на разных портах
- Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
- HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

### Решение 2

Добавляем домен `example.local` в `/etc/hosts` чтобы VM знала, какой IP соответствует доменному имени.

```
sudo nano /etc/hosts
```
добавляем строку (IP VM)
```
192.168.1.227   example.local
```

`haproxy.cfg`

```
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermedia>
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECD>
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POL>
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5s
        timeout client  50s
        timeout server  50s
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

listen stats   
        bind *:888
        mode http
        option httplog
        stats enable
        stats uri /stats
        stats refresh 5s
        stats realm Haproxy\ Statistics
        #stats auth admin:admin

frontend example   
        mode http
        bind :8088
        option httplog
        acl ACL_example hdr_beg(host) -i example.local
        use_backend web_cluster if ACL_example
        default_backend reject_all

backend web_cluster # backend for example.local
        mode http
        balance roundrobin
        ### Weighted Round Robin ###
        server s1 127.0.0.1:8888 weight 2 check
        server s2 127.0.0.1:9999 weight 3 check
        server s3 127.0.0.1:7777 weight 4 check

backend reject_all # backend by default
        mode http
        http-request deny deny_status 403
```

`Cкриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него:`

<img width="725" height="547" alt="Capture5" src="https://github.com/user-attachments/assets/74e83b4f-c748-4e97-a4c5-601be7674bbc" />

<img width="1032" height="690" alt="Capture6" src="https://github.com/user-attachments/assets/d6fe2c5d-c16f-4537-8c7e-3d783f891d45" />
