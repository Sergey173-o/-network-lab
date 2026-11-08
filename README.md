
## Схема


````mermaid
flowchart TD
    I[Интернет] --- IR[inetRouter]
    IR ---|192.168.255.0/30| CR[centralRouter]
    CR ---|192.168.255.8/30| O1[office1Router]
    CR ---|192.168.255.4/30| O2[office2Router]
    CR ---|Directors| CS[centralServer]
    O1 ---|Managers| S1[office1Server]
    O2 ---|Dev| S2[office2Server]
````
## Адресация

Вместимость — число адресов между адресом сети и broadcast, включая адрес шлюза. Все сети начинаются на границах своих префиксов и не пересекаются. Office1 и Office2 полностью занимают свои блоки /24. Для Office2 Dev используется /25

````bash
| Сегмент | Подсеть | Узлов | Broadcast |
|---|---|---:|---|
| Central · Directors | `192.168.0.0/28` | 14 | `192.168.0.15` |
| Central · Office Hardware | `192.168.0.32/28` | 14 | `192.168.0.47` |
| Central · Wi-Fi | `192.168.0.64/26` | 62 | `192.168.0.127` |
| Office1 · Dev | `192.168.2.0/26` | 62 | `192.168.2.63` |
| Office1 · Test Servers | `192.168.2.64/26` | 62 | `192.168.2.127` |
| Office1 · Managers | `192.168.2.128/26` | 62 | `192.168.2.191` |
| Office1 · Office Hardware | `192.168.2.192/26` | 62 | `192.168.2.255` |
| Office2 · Dev | `192.168.1.0/25` | 126 | `192.168.1.127` |
| Office2 · Test Servers | `192.168.1.128/26` | 62 | `192.168.1.191` |
| Office2 · Office Hardware | `192.168.1.192/26` | 62 | `192.168.1.255` |
| Inet ↔ Central | `192.168.255.0/30` | 2 | `192.168.255.3` |
| Central ↔ Office2 | `192.168.255.4/30` | 2 | `192.168.255.7` |
| Central ↔ Office1 | `192.168.255.8/30` | 2 | `192.168.255.11` |

Свободные блоки в выделенных /24 после добавления двух транзитных линков.

| Подсеть | Узлов | Broadcast |
|---|---:|---|
| `192.168.0.16/28` | 14 | `192.168.0.31` |
| `192.168.0.48/28` | 14 | `192.168.0.63` |
| `192.168.0.128/25` | 126 | `192.168.0.255` |
| `192.168.255.12/30` | 2 | `192.168.255.15` |
| `192.168.255.16/28` | 14 | `192.168.255.31` |
| `192.168.255.32/27` | 30 | `192.168.255.63` |
| `192.168.255.64/26` | 62 | `192.168.255.127` |
| `192.168.255.128/25` | 126 | `192.168.255.255` |

## Запуск

   ```bash
root@nfsc:~/network-lab-console# for vm in centralRouter office1Router office2Router centralServer office1Server; do
  vagrant provision "$vm" || break
done

for vm in inetRouter centralRouter office1Router office2Router centralServer office1Server office2Server; do
  echo "=== $vm ==="
  vagrant ssh "$vm" -c 'timeout 15 getent ahostsv4 example.com' ||
    echo "Ошибка DNS или SSH на $vm"
done
==> centralRouter: You assigned a static IP ending in ".1" or ":1" to this machine.
==> centralRouter: This is very often used by the router and can cause the
==> centralRouter: network to not work properly. If the network doesn't work
==> centralRouter: properly, try changing this IP.
==> centralRouter: Running provisioner: ansible...
    centralRouter: Running ansible-playbook...

PLAY [Настроить сетевую лабораторию] *******************************************

TASK [Gathering Facts] *********************************************************
ok: [centralRouter]

TASK [Создать каталог конфигурации резолвера] **********************************
ok: [centralRouter]

TASK [Сохранить настройку DNSSEC для лаборатории] ******************************
changed: [centralRouter]

TASK [Применить настройки резолвера] *******************************************
changed: [centralRouter]

TASK [Создать каталог настроек systemd-resolved] *******************************
ok: [centralRouter]

TASK [Отключить локальную проверку DNSSEC в лаборатории] ***********************
ok: [centralRouter]

TASK [Перезапустить systemd-resolved после изменения] **************************
skipping: [centralRouter]

TASK [Проверить сетевой сервис Ubuntu] *****************************************
ok: [centralRouter]

TASK [Сохранить настройку ядра] ************************************************
ok: [centralRouter]

TASK [Применить настройку ядра] ************************************************
ok: [centralRouter]

TASK [Настроить адреса и маршруты на внутренних интерфейсах] *******************
ok: [centralRouter] => (item=inet-central)
ok: [centralRouter] => (item=directors)
ok: [centralRouter] => (item=central-hardware)
ok: [centralRouter] => (item=wifi)
ok: [centralRouter] => (item=central-office1)
ok: [centralRouter] => (item=central-office2)

TASK [Оставить DHCP на NAT, отключив маршруты и DNS от него] *******************
ok: [centralRouter]

TASK [Создать каталог настроек DNS] ********************************************
skipping: [centralRouter]

TASK [Настроить DNS VirtualBox на inetRouter] **********************************
skipping: [centralRouter]

TASK [Применить настройки DNS] *************************************************
skipping: [centralRouter]

TASK [Установить nftables] *****************************************************
skipping: [centralRouter]

TASK [Сохранить правила фильтрации и NAT] **************************************
skipping: [centralRouter]

TASK [Включить и загрузить правила nftables] ***********************************
skipping: [centralRouter]

TASK [Перечитать конфигурацию интерфейсов] *************************************
ok: [centralRouter]

PLAY RECAP *********************************************************************
centralRouter              : ok=12   changed=2    unreachable=0    failed=0    skipped=7    rescued=0    ignored=0

==> office1Router: You assigned a static IP ending in ".1" or ":1" to this machine.
==> office1Router: This is very often used by the router and can cause the
==> office1Router: network to not work properly. If the network doesn't work
==> office1Router: properly, try changing this IP.
==> office1Router: Running provisioner: ansible...
    office1Router: Running ansible-playbook...

PLAY [Настроить сетевую лабораторию] *******************************************

TASK [Gathering Facts] *********************************************************
ok: [office1Router]

TASK [Создать каталог конфигурации резолвера] **********************************
changed: [office1Router]

TASK [Сохранить настройку DNSSEC для лаборатории] ******************************
changed: [office1Router]

TASK [Применить настройки резолвера] *******************************************
changed: [office1Router]

TASK [Создать каталог настроек systemd-resolved] *******************************
skipping: [office1Router]

TASK [Отключить локальную проверку DNSSEC в лаборатории] ***********************
skipping: [office1Router]

TASK [Перезапустить systemd-resolved после изменения] **************************
skipping: [office1Router]

TASK [Проверить сетевой сервис Ubuntu] *****************************************
ok: [office1Router]

TASK [Сохранить настройку ядра] ************************************************
ok: [office1Router]

TASK [Применить настройку ядра] ************************************************
ok: [office1Router]

TASK [Настроить адреса и маршруты на внутренних интерфейсах] *******************
ok: [office1Router] => (item=central-office1)
ok: [office1Router] => (item=office1-dev)
ok: [office1Router] => (item=office1-test)
ok: [office1Router] => (item=office1-managers)
ok: [office1Router] => (item=office1-hardware)

TASK [Оставить DHCP на NAT, отключив маршруты и DNS от него] *******************
ok: [office1Router]

TASK [Создать каталог настроек DNS] ********************************************
skipping: [office1Router]

TASK [Настроить DNS VirtualBox на inetRouter] **********************************
skipping: [office1Router]

TASK [Применить настройки DNS] *************************************************
skipping: [office1Router]

TASK [Установить nftables] *****************************************************
skipping: [office1Router]

TASK [Сохранить правила фильтрации и NAT] **************************************
skipping: [office1Router]

TASK [Включить и загрузить правила nftables] ***********************************
skipping: [office1Router]

TASK [Перечитать конфигурацию интерфейсов] *************************************
ok: [office1Router]

PLAY RECAP *********************************************************************
office1Router              : ok=10   changed=3    unreachable=0    failed=0    skipped=9    rescued=0    ignored=0

==> office2Router: You assigned a static IP ending in ".1" or ":1" to this machine.
==> office2Router: This is very often used by the router and can cause the
==> office2Router: network to not work properly. If the network doesn't work
==> office2Router: properly, try changing this IP.
==> office2Router: Running provisioner: ansible...
    office2Router: Running ansible-playbook...

PLAY [Настроить сетевую лабораторию] *******************************************

TASK [Gathering Facts] *********************************************************
ok: [office2Router]

TASK [Создать каталог конфигурации резолвера] **********************************
changed: [office2Router]

TASK [Сохранить настройку DNSSEC для лаборатории] ******************************
changed: [office2Router]

TASK [Применить настройки резолвера] *******************************************
changed: [office2Router]

TASK [Создать каталог настроек systemd-resolved] *******************************
skipping: [office2Router]

TASK [Отключить локальную проверку DNSSEC в лаборатории] ***********************
skipping: [office2Router]

TASK [Перезапустить systemd-resolved после изменения] **************************
skipping: [office2Router]

TASK [Проверить сетевой сервис Ubuntu] *****************************************
ok: [office2Router]

TASK [Сохранить настройку ядра] ************************************************
ok: [office2Router]

TASK [Применить настройку ядра] ************************************************
ok: [office2Router]

TASK [Настроить адреса и маршруты на внутренних интерфейсах] *******************
ok: [office2Router] => (item=central-office2)
ok: [office2Router] => (item=office2-dev)
ok: [office2Router] => (item=office2-test)
ok: [office2Router] => (item=office2-hardware)

TASK [Оставить DHCP на NAT, отключив маршруты и DNS от него] *******************
ok: [office2Router]

TASK [Создать каталог настроек DNS] ********************************************
skipping: [office2Router]

TASK [Настроить DNS VirtualBox на inetRouter] **********************************
skipping: [office2Router]

TASK [Применить настройки DNS] *************************************************
skipping: [office2Router]

TASK [Установить nftables] *****************************************************
skipping: [office2Router]

TASK [Сохранить правила фильтрации и NAT] **************************************
skipping: [office2Router]

TASK [Включить и загрузить правила nftables] ***********************************
skipping: [office2Router]

TASK [Перечитать конфигурацию интерфейсов] *************************************
ok: [office2Router]

PLAY RECAP *********************************************************************
office2Router              : ok=10   changed=3    unreachable=0    failed=0    skipped=9    rescued=0    ignored=0

==> centralServer: Running provisioner: ansible...
    centralServer: Running ansible-playbook...

PLAY [Настроить сетевую лабораторию] *******************************************

TASK [Gathering Facts] *********************************************************
ok: [centralServer]

TASK [Создать каталог конфигурации резолвера] **********************************
changed: [centralServer]

TASK [Сохранить настройку DNSSEC для лаборатории] ******************************
changed: [centralServer]

TASK [Применить настройки резолвера] *******************************************
changed: [centralServer]

TASK [Создать каталог настроек systemd-resolved] *******************************
skipping: [centralServer]

TASK [Отключить локальную проверку DNSSEC в лаборатории] ***********************
skipping: [centralServer]

TASK [Перезапустить systemd-resolved после изменения] **************************
skipping: [centralServer]

TASK [Проверить сетевой сервис Ubuntu] *****************************************
ok: [centralServer]

TASK [Сохранить настройку ядра] ************************************************
skipping: [centralServer]

TASK [Применить настройку ядра] ************************************************
skipping: [centralServer]

TASK [Настроить адреса и маршруты на внутренних интерфейсах] *******************
ok: [centralServer] => (item=directors)

TASK [Оставить DHCP на NAT, отключив маршруты и DNS от него] *******************
ok: [centralServer]

TASK [Создать каталог настроек DNS] ********************************************
skipping: [centralServer]

TASK [Настроить DNS VirtualBox на inetRouter] **********************************
skipping: [centralServer]

TASK [Применить настройки DNS] *************************************************
skipping: [centralServer]

TASK [Установить nftables] *****************************************************
skipping: [centralServer]

TASK [Сохранить правила фильтрации и NAT] **************************************
skipping: [centralServer]

TASK [Включить и загрузить правила nftables] ***********************************
skipping: [centralServer]

TASK [Перечитать конфигурацию интерфейсов] *************************************
ok: [centralServer]

PLAY RECAP *********************************************************************
centralServer              : ok=8    changed=3    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0

==> office1Server: Running provisioner: ansible...
    office1Server: Running ansible-playbook...

PLAY [Настроить сетевую лабораторию] *******************************************

TASK [Gathering Facts] *********************************************************
ok: [office1Server]

TASK [Создать каталог конфигурации резолвера] **********************************
changed: [office1Server]

TASK [Сохранить настройку DNSSEC для лаборатории] ******************************
changed: [office1Server]

TASK [Применить настройки резолвера] *******************************************
changed: [office1Server]

TASK [Создать каталог настроек systemd-resolved] *******************************
skipping: [office1Server]

TASK [Отключить локальную проверку DNSSEC в лаборатории] ***********************
skipping: [office1Server]

TASK [Перезапустить systemd-resolved после изменения] **************************
skipping: [office1Server]

TASK [Проверить сетевой сервис Ubuntu] *****************************************
ok: [office1Server]

TASK [Сохранить настройку ядра] ************************************************
skipping: [office1Server]

TASK [Применить настройку ядра] ************************************************
skipping: [office1Server]

TASK [Настроить адреса и маршруты на внутренних интерфейсах] *******************
ok: [office1Server] => (item=office1-managers)

TASK [Оставить DHCP на NAT, отключив маршруты и DNS от него] *******************
ok: [office1Server]

TASK [Создать каталог настроек DNS] ********************************************
skipping: [office1Server]

TASK [Настроить DNS VirtualBox на inetRouter] **********************************
skipping: [office1Server]

TASK [Применить настройки DNS] *************************************************
skipping: [office1Server]

TASK [Установить nftables] *****************************************************
skipping: [office1Server]

TASK [Сохранить правила фильтрации и NAT] **************************************
skipping: [office1Server]

TASK [Включить и загрузить правила nftables] ***********************************
skipping: [office1Server]

TASK [Перечитать конфигурацию интерфейсов] *************************************
ok: [office1Server]

PLAY RECAP *********************************************************************
office1Server              : ok=8    changed=3    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0

=== inetRouter ===
172.66.147.243  STREAM example.com
172.66.147.243  DGRAM
172.66.147.243  RAW
104.20.23.154   STREAM
104.20.23.154   DGRAM
104.20.23.154   RAW
=== centralRouter ===
8.47.69.0       STREAM example.com
8.47.69.0       DGRAM
8.47.69.0       RAW
8.6.112.0       STREAM
8.6.112.0       DGRAM
8.6.112.0       RAW
=== office1Router ===
8.47.69.0       STREAM example.com
8.47.69.0       DGRAM
8.47.69.0       RAW
8.6.112.0       STREAM
8.6.112.0       DGRAM
8.6.112.0       RAW
=== office2Router ===
8.47.69.0       STREAM example.com
8.47.69.0       DGRAM
8.47.69.0       RAW
8.6.112.0       STREAM
8.6.112.0       DGRAM
8.6.112.0       RAW
=== centralServer ===
8.47.69.0       STREAM example.com
8.47.69.0       DGRAM
8.47.69.0       RAW
8.6.112.0       STREAM
8.6.112.0       DGRAM
8.6.112.0       RAW
=== office1Server ===
8.6.112.0       STREAM example.com
8.6.112.0       DGRAM
8.6.112.0       RAW
8.47.69.0       STREAM
8.47.69.0       DGRAM
8.47.69.0       RAW
=== office2Server ===
8.6.112.0       STREAM example.com
8.6.112.0       DGRAM
8.6.112.0       RAW
8.47.69.0       STREAM
8.47.69.0       DGRAM
8.47.69.0       RAW
root@nfsc:~/network-lab-console#

````

## Проверка


маршруты по умолчанию и HTTPS со всех ВМ

```bash
root@nfsc:~/network-lab-console# cd ~/network-lab-console
vagrant status

for vm in inetRouter centralRouter office1Router office2Router centralServer office1Server office2Server; do
  echo "=== $vm ==="
  vagrant ssh "$vm" -c 'ip -4 route show default; timeout 15s curl -4fsSI --noproxy "*" https://example.com' ||
    echo "Ошибка HTTPS или SSH на $vm"
done

vagrant ssh inetRouter -c 'sudo nft list chain ip network_lab postrouting'
Current machine states:

inetRouter                running (virtualbox)
centralRouter             running (virtualbox)
office1Router             running (virtualbox)
office2Router             running (virtualbox)
centralServer             running (virtualbox)
office1Server             running (virtualbox)
office2Server             running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
=== inetRouter ===
default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 metric 100
HTTP/2 200
date: Sun, 08 Nov 2026 18:11:40 GMT
content-type: text/html; charset=utf-8
server: cloudflare
last-modified: Wed, 04 Nov 2026 18:12:44 GMT
allow: GET, HEAD
accept-ranges: bytes
age: 7118
cf-cache-status: HIT
cf-ray: a5768aa20876a1aa-TLL
alt-svc: h3=":443"; ma=86400

=== centralRouter ===
default via 192.168.255.1 dev eth1 proto static
HTTP/2 200
date: Sun, 08 Nov 2026 18:11:56 GMT
content-type: text/html; charset=utf-8
server: cloudflare
last-modified: Wed, 04 Nov 2026 18:12:44 GMT
allow: GET, HEAD
accept-ranges: bytes
age: 8582
cf-cache-status: HIT
cf-ray: a5768b04fb4e96ea-HEL
alt-svc: h3=":443"; ma=86400

=== office1Router ===
default via 192.168.255.9 dev eth1 proto static
HTTP/2 200
date: Sun, 08 Nov 2026 18:12:14 GMT
content-type: text/html; charset=utf-8
server: cloudflare
last-modified: Wed, 04 Nov 2026 18:12:44 GMT
allow: GET, HEAD
accept-ranges: bytes
age: 4
cf-cache-status: HIT
cf-ray: a5768b7819cc588e-ARN
alt-svc: h3=":443"; ma=86400

=== office2Router ===
default via 192.168.255.5 dev eth1 proto static
HTTP/2 200
date: Sun, 08 Nov 2026 18:12:42 GMT
content-type: text/html; charset=utf-8
server: cloudflare
last-modified: Wed, 04 Nov 2026 18:12:44 GMT
allow: GET, HEAD
accept-ranges: bytes
age: 7
cf-cache-status: HIT
cf-ray: a5768c235cff8c4a-ARN
alt-svc: h3=":443"; ma=86400

=== centralServer ===
default via 192.168.0.1 dev eth1 proto static
HTTP/2 200
date: Sun, 08 Nov 2026 18:12:56 GMT
content-type: text/html; charset=utf-8
server: cloudflare
last-modified: Wed, 04 Nov 2026 18:12:44 GMT
allow: GET, HEAD
accept-ranges: bytes
age: 8642
cf-cache-status: HIT
cf-ray: a5768c7d4f187601-HEL
alt-svc: h3=":443"; ma=86400

=== office1Server ===
default via 192.168.2.129 dev eth1 proto static
HTTP/2 200
date: Sun, 08 Nov 2026 18:13:12 GMT
content-type: text/html; charset=utf-8
server: cloudflare
last-modified: Wed, 04 Nov 2026 18:12:44 GMT
allow: GET, HEAD
accept-ranges: bytes
age: 8
cf-cache-status: HIT
cf-ray: a5768ce02e3a13b8-ARN
alt-svc: h3=":443"; ma=86400

=== office2Server ===
default via 192.168.1.1 dev eth1 proto static
HTTP/2 200
date: Sun, 08 Nov 2026 18:13:38 GMT
content-type: text/html; charset=utf-8
server: cloudflare
last-modified: Wed, 04 Nov 2026 18:12:44 GMT
allow: GET, HEAD
accept-ranges: bytes
age: 8685
cf-cache-status: HIT
cf-ray: a5768d853f960fb4-HEL
alt-svc: h3=":443"; ma=86400

table ip network_lab {
        chain postrouting {
                type nat hook postrouting priority srcnat; policy accept;
                oifname "eth0" ip saddr { 192.168.0.0-192.168.2.255, 192.168.255.0/24 } counter packets 356 bytes 30856 masquerade
        }
}
root@nfsc:~/network-lab-console#

 ```

проверка связности между всеми ВМ.

   ```bash
  root@nfsc:~/network-lab-console# vms=(inetRouter centralRouter office1Router office2Router centralServer office1Server office2Server)
ips=(192.168.255.1 192.168.255.2 192.168.255.10 192.168.255.6 192.168.0.2 192.168.2.130 192.168.1.2)

for i in "${!vms[@]}"; do
  check=true
  for j in "${!ips[@]}"; do
    [[ "$i" == "$j" ]] && continue
    check+=" && (ping -c 2 -W 2 ${ips[j]} >/dev/null || { echo 'Нет ответа от ${ips[j]}'; exit 1; })"
  done
  if vagrant ssh "${vms[i]}" -c "$check"; then
    echo "${vms[i]}: 6 из 6 доступны"
  else
    echo "${vms[i]}: проверка не пройдена"
  fi
done
inetRouter: 6 из 6 доступны
centralRouter: 6 из 6 доступны
office1Router: 6 из 6 доступны
office2Router: 6 из 6 доступны
centralServer: 6 из 6 доступны
office1Server: 6 из 6 доступны
office2Server: 6 из 6 доступны
root@nfsc:~/network-lab-console#


   ```
