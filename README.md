# VPN-Site-To-Site---Fortigate

# 🔐 VPN Site-To-Site con FortiGate en GNS3

## 📌 Objetivo

En esta práctica implementé una **VPN Site-To-Site** utilizando dispositivos FortiGate en GNS3, con el objetivo de permitir la comunicación segura entre dos redes LAN remotas a través de un enlace simulado de Internet.

Con esta configuración logré:

* Conectar dos redes privadas
* Proteger la comunicación mediante IPsec
* Verificar conectividad con ping y traceroute

---

## 🧩 Topología

La topología utilizada fue:

```
PC1 --- FG1 --- ISP --- FG2 --- PC2
```

* El router ISP simula Internet
* Cada FortiGate protege una red LAN

---

## 🌐 Direccionamiento IP

### 🔹 LAN 1 (Sitio A)

* Red: 12.0.0.0/24
* PC1: 12.0.0.10
* Gateway: 12.0.0.1

### 🔹 LAN 2 (Sitio B)

* Red: 12.0.1.0/24
* PC2: 12.0.1.10
* Gateway: 12.0.1.1

---

### 🔹 FortiGate 1 (FG1)

* port1 (LAN): 12.0.0.1/24
* port2 (WAN): 100.0.0.2/30

---

### 🔹 FortiGate 2 (FG2)

* port1 (LAN): 12.0.1.1/24
* port2 (WAN): 100.0.0.6/30

---

### 🔹 Router ISP

* e0/0: 100.0.0.1/30
* e0/1: 100.0.0.5/30

---

## ⚙️ Configuración

---

### 🔹 Router ISP

```bash
enable
configure terminal

interface e0/0
ip address 100.0.0.1 255.255.255.252
no shutdown

interface e0/1
ip address 100.0.0.5 255.255.255.252
no shutdown

ip route 12.0.0.0 255.255.255.0 100.0.0.2
ip route 12.0.1.0 255.255.255.0 100.0.0.6

end
```

---

### 🔹 FortiGate 1 (FG1)

#### Interfaces

```bash
config system interface

edit port1
set mode static
set ip 12.0.0.1/24
set allowaccess ping https ssh
next

edit port2
set mode static
set ip 100.0.0.2/30
set allowaccess ping https ssh
next

end
```

---

#### Ruta por defecto

```bash
config router static
edit 1
set dst 0.0.0.0/0
set gateway 100.0.0.1
set device port2
next
end
```

---

#### VPN Fase 1

```bash
config vpn ipsec phase1-interface
edit "VPN-SITE"
set interface "port2"
set remote-gw 100.0.0.6
set psksecret vpn123
next
end
```

---

#### VPN Fase 2

```bash
config vpn ipsec phase2-interface
edit "VPN-SITE-P2"
set phase1name "VPN-SITE"
set src-subnet 12.0.0.0 255.255.255.0
set dst-subnet 12.0.1.0 255.255.255.0
next
end
```

---

#### Políticas

```bash
config firewall policy

edit 1
set name "LAN_to_VPN"
set srcintf "port1"
set dstintf "VPN-SITE"
set srcaddr "all"
set dstaddr "all"
set action accept
set schedule always
set service ALL
set nat disable
next

edit 2
set name "VPN_to_LAN"
set srcintf "VPN-SITE"
set dstintf "port1"
set srcaddr "all"
set dstaddr "all"
set action accept
set schedule always
set service ALL
set nat disable
next

end
```

---

### 🔹 FortiGate 2 (FG2)

#### Interfaces

```bash
config system interface

edit port1
set mode static
set ip 12.0.1.1/24
set allowaccess ping https ssh
next

edit port2
set mode static
set ip 100.0.0.6/30
set allowaccess ping https ssh
next

end
```

---

#### Ruta por defecto

```bash
config router static
edit 1
set dst 0.0.0.0/0
set gateway 100.0.0.5
set device port2
next
end
```

---

#### VPN Fase 1

```bash
config vpn ipsec phase1-interface
edit "VPN-SITE"
set interface "port2"
set remote-gw 100.0.0.2
set psksecret vpn123
next
end
```

---

#### VPN Fase 2

```bash
config vpn ipsec phase2-interface
edit "VPN-SITE-P2"
set phase1name "VPN-SITE"
set src-subnet 12.0.1.0 255.255.255.0
set dst-subnet 12.0.0.0 255.255.255.0
next
end
```

---

#### Políticas

```bash
config firewall policy

edit 1
set name "LAN_to_VPN"
set srcintf "port1"
set dstintf "VPN-SITE"
set srcaddr "all"
set dstaddr "all"
set action accept
set schedule always
set service ALL
set nat disable
next

edit 2
set name "VPN_to_LAN"
set srcintf "VPN-SITE"
set dstintf "port1"
set srcaddr "all"
set dstaddr "all"
set action accept
set schedule always
set service ALL
set nat disable
next

end
```

---

## 🧪 Pruebas

### 🔹 Antes de la VPN

* Ping entre WANs: ✅
* Ping entre LANs: ❌

---

### 🔹 Después de la VPN

```bash
ping 12.0.1.10
```

Resultado:

* Comunicación exitosa entre redes

---

### 🔹 Traceroute

```bash
tracert 12.0.1.10
```

Ruta observada:

```
PC1 → FG1 → VPN → FG2 → PC2
```

---

## 📸 Evidencia requerida

Incluí capturas de:

* Topología en GNS3
* Configuración de interfaces
* Fase 1 y Fase 2
* Políticas de firewall
* Ping exitoso
* Traceroute
* Estado del túnel:

```bash
diagnose vpn tunnel list
```

---

## ✅ Conclusión

En esta práctica logré implementar una VPN Site-To-Site funcional utilizando FortiGate, permitiendo la comunicación segura entre dos redes remotas mediante un túnel IPsec.

Aprendí la importancia de:

* Configurar correctamente las fases de la VPN
* Aplicar políticas en ambos sentidos
* Verificar el estado del túnel para asegurar la conectividad
