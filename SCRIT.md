# IPSec IKEv1
## Configure una VPN site to site punto a punto con túnel GRE

---

## Paso 1 — Configuración de interfaces y ruta por defecto

**R2**
```
conf t
interface e0/0
 ip address 200.1.15.2 255.255.255.0
 no shut
interface e0/1
 ip address 10.15.99.1 255.255.255.0
 no shut
ip route 0.0.0.0 0.0.0.0 200.1.15.1
```

**R3**
```
conf t
interface e0/0
 ip address 200.1.99.2 255.255.255.0
 no shut
interface e0/1
 ip address 192.168.99.1 255.255.255.0
 no shut
ip route 0.0.0.0 0.0.0.0 200.1.99.1
```

**R1**
```
conf t
interface e0/0
 ip address 200.1.15.1 255.255.255.0
 no shut
interface e0/1
 ip address 200.1.99.1 255.255.255.0
 no shut
```

---

## Paso 2 — IKEv1 Fase 1 (ISAKMP policy)

**R2**
```
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
!
crypto isakmp key cisco address 200.1.99.2
```

**R3**
```
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
!
crypto isakmp key cisco address 200.1.15.2
```

---

## Paso 3 — IPSec Fase 2 (transform-set en modo transporte)

**R2**
```
crypto ipsec transform-set GRE-SET esp-aes esp-sha-hmac
 mode transport
```

**R3**
```
crypto ipsec transform-set GRE-SET esp-aes esp-sha-hmac
 mode transport
```

---

## Paso 4 — Interfaz Túnel GRE

**R2**
```
interface Tunnel0
 ip address 172.16.1.1 255.255.255.252
 tunnel source 200.1.15.2
 tunnel destination 200.1.99.2
 tunnel mode gre ip
 tunnel protection ipsec profile GRE-PROFILE
```

**R3**
```
interface Tunnel0
 ip address 172.16.1.2 255.255.255.252
 tunnel source 200.1.99.2
 tunnel destination 200.1.15.2
 tunnel mode gre ip
 tunnel protection ipsec profile GRE-PROFILE
```

---

## Paso 5 — Perfil IPSec (vincula transform-set al túnel)

**R2**
```
crypto ipsec profile GRE-PROFILE
 set transform-set GRE-SET
```

**R3**
```
crypto ipsec profile GRE-PROFILE
 set transform-set GRE-SET
```

---

## Paso 6 — Rutas estáticas por el túnel GRE

**R2**
```
ip route 192.168.99.0 255.255.255.0 Tunnel0
```

**R3**
```
ip route 10.15.99.0 255.255.255.0 Tunnel0
```

**VPC8**
```
ip 10.15.99.3 255.255.255.0 10.15.99.1
```

**VPC9**
```
ip 192.168.99.3 255.255.255.0 192.168.99.1
```
