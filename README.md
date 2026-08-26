# Implementación de VPN IPSec Site-to-Site con Cisco Packet Tracer

## Descripción del proyecto

Este proyecto consiste en la implementación de una conexión **VPN IPSec Site-to-Site** utilizando routers Cisco 2911 dentro de un entorno simulado en **Cisco Packet Tracer**.

El objetivo principal es establecer una comunicación segura entre dos redes LAN independientes mediante un túnel VPN cifrado utilizando el protocolo IPSec.

La arquitectura implementada permite que dos redes privadas puedan comunicarse a través de una red ISP simulada, protegiendo la información mediante mecanismos de cifrado, autenticación e integridad.

---

# Topología de red

La infraestructura está conformada por:

- Router R1 (Sitio A)
- Router ISP (Proveedor de Internet simulado)
- Router R2 (Sitio B)
- Switch SW1
- Switch SW2
- PC0
- Server0

La comunicación se establece mediante:

- LAN Sitio A: 192.168.10.0/24
- WAN R1-ISP: 200.0.0.0/30
- WAN ISP-R2: 201.0.0.0/30
- LAN Sitio B: 192.168.20.0/24


<img width="789" height="446" alt="image" src="https://github.com/user-attachments/assets/ae280922-39b3-41f1-b8eb-b471f15b4ba3" />


---

# Direccionamiento IP

## Red LAN Sitio A

| Dispositivo | Interfaz | Dirección IP |
|-------------|----------|--------------|
| R1 | GigabitEthernet0/0 | 192.168.10.1/24 |
| PC0 | FastEthernet | 192.168.10.X/24 |

## Enlace R1 - ISP

| Dispositivo | Interfaz | Dirección IP |
|-------------|----------|--------------|
| R1 | Serial0/3/0 | 200.0.0.1/30 |
| ISP | Serial0/3/0 | 200.0.0.2/30 |

## Enlace ISP - R2

| Dispositivo | Interfaz | Dirección IP |
|-------------|----------|--------------|
| ISP | Serial0/3/1 | 201.0.0.1/30 |
| R2 | Serial0/3/0 | 201.0.0.2/30 |

## Red LAN Sitio B

| Dispositivo | Interfaz | Dirección IP |
|-------------|----------|--------------|
| R2 | GigabitEthernet0/0 | 192.168.20.1/24 |
| Server0 | FastEthernet | 192.168.20.X/24 |

---

# Verificación de interfaces

Para comprobar que las interfaces se encuentran correctamente configuradas se utilizó:

```bash
show ip interface brief
```

## Router R1

<img width="858" height="158" alt="image" src="https://github.com/user-attachments/assets/0212c15d-aac6-4fa0-9770-681b5e0f1de8" />

## Router ISP

<img width="869" height="144" alt="image" src="https://github.com/user-attachments/assets/593ff7a9-e2f6-4f82-8d90-bdfed85a9bc2" />

## Router R2

<img width="861" height="146" alt="image" src="https://github.com/user-attachments/assets/93852555-b677-449c-951b-5aeef36de63d" />

---

# Configuración de enrutamiento

Se utilizaron rutas estáticas para permitir comunicación entre ambas redes LAN.

## Router R1

```bash
ip route 192.168.20.0 255.255.255.0 200.0.0.2
ip route 201.0.0.0 255.255.255.252 200.0.0.2
```

## Router ISP

```bash
ip route 192.168.10.0 255.255.255.0 200.0.0.1
ip route 192.168.20.0 255.255.255.0 201.0.0.2
```

## Router R2

```bash
ip route 192.168.10.0 255.255.255.0 201.0.0.1
ip route 200.0.0.0 255.255.255.252 201.0.0.1
```

---

# Verificación de tablas de rutas

Comando utilizado:

```bash
show ip route
```

## Tabla de rutas R1

<img width="864" height="335" alt="{89C42635-5582-4AC2-821E-A28DDA8AC006}" src="https://github.com/user-attachments/assets/75c4205e-adfa-4f2a-981d-03235dd4b175" />

## Tabla de rutas ISP

<img width="864" height="319" alt="{B5683917-C461-4AD2-8585-597EDC1264F4}" src="https://github.com/user-attachments/assets/afdcee70-9816-4842-b64e-e0e0ac5baafe" />

## Tabla de rutas R2

<img width="860" height="349" alt="{D76CF1CB-0532-4838-80E4-2E2640F2D2A8}" src="https://github.com/user-attachments/assets/edbc4719-0d72-4a30-a258-5f14d94a962c" />

---

# Configuración VPN IPSec

La VPN fue implementada utilizando:

- IPSec Site-to-Site
- IKE Phase 1
- IPSec Phase 2
- AES como algoritmo de cifrado
- SHA-HMAC para integridad
- Clave precompartida


## Parámetros de seguridad

| Parámetro | Valor |
|-----------|-------|
| Protocolo | IPSec |
| Cifrado | AES |
| Autenticación | Pre-Shared Key |
| Grupo Diffie-Hellman | Grupo 2 |
| Hash | SHA-HMAC |
| Clave compartida | CLAVE123 |

---

# Configuración ISAKMP
<img width="852" height="217" alt="{6AE23FD7-9475-42C3-BEF3-0AAE9C2B0474}" src="https://github.com/user-attachments/assets/f1032868-7c80-494a-bc27-e095470cf38b" />
<img width="858" height="190" alt="{2E76C5AF-F021-4426-87B9-3B10BE72B6ED}" src="https://github.com/user-attachments/assets/2df78e53-a7b0-431c-afc8-c8bf0d3851d5" />

En ambos routers:

```bash
crypto isakmp policy 10
 encryption aes
 authentication pre-share
 group 2
```

---

# Configuración IPSec Transform Set

```bash
crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
```

---

# Configuración Crypto Map

## R1

```bash
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 201.0.0.2
 set transform-set VPN-SET
 match address 100

interface Serial0/3/0
 crypto map VPN-MAP
```

## R2

```bash
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 200.0.0.1
 set transform-set VPN-SET
 match address 100

interface Serial0/3/0
 crypto map VPN-MAP
```

---

# Verificación del túnel VPN

## Estado IKE

Comando:

```bash
show crypto isakmp sa
```

<img width="859" height="162" alt="{595EFFD1-C7F7-4045-85A3-2A846E9C566E}" src="https://github.com/user-attachments/assets/45270580-26d1-4baf-ab35-5d638c9bb055" />
<img width="848" height="155" alt="{4044EC21-79CE-4E48-A9A8-A728DCAF4B3A}" src="https://github.com/user-attachments/assets/a6f3442c-024f-4d17-af79-fead8429b35b" />

El estado esperado es:

```
QM_IDLE
```

lo cual indica que la negociación IKE fue exitosa.

---

## Estado IPSec

Comando:

```bash
show crypto ipsec sa
```

<img width="863" height="244" alt="{CA25737E-4511-4226-86C5-6858ABF7D845}" src="https://github.com/user-attachments/assets/97a17deb-8143-41b9-8b58-4d81d011be77" />
<img width="860" height="368" alt="{3C4D317F-FAA5-4938-BC48-004D9445CE23}" src="https://github.com/user-attachments/assets/6ad7bf17-5b34-4cbf-8922-2ed851baf6d2" />

Se verifican los contadores:

```
#pkts encaps
#pkts encrypt
#pkts decaps
#pkts decrypt
```

Estos valores demuestran que existe tráfico cifrado mediante el túnel VPN.

---

# Pruebas de conectividad

Se realizaron pruebas ICMP entre ambas redes.

## R1 hacia R2

```bash
ping 192.168.20.1
```

<img width="857" height="133" alt="{B458792A-ADEE-46B7-8A40-85D27A59515F}" src="https://github.com/user-attachments/assets/0745945b-4f49-405d-83ab-3be51057374b" />

## R2 hacia R1

```bash
ping 192.168.10.1
```

<img width="859" height="136" alt="{4531F146-1D65-411C-B6C9-61E0D611C550}" src="https://github.com/user-attachments/assets/08c28249-e638-4bb4-80db-69e57c50bcbd" />


Resultado:

```
Success rate is 100 percent
```

---

# Conclusión

La implementación de la VPN IPSec Site-to-Site fue realizada correctamente utilizando routers Cisco 2911.

Se logró establecer una comunicación segura entre dos redes LAN mediante un túnel cifrado utilizando IPSec, garantizando autenticación, confidencialidad e integridad de la información.

Las pruebas realizadas confirmaron:

- Correcta configuración de interfaces.
- Comunicación entre redes mediante rutas estáticas.
- Negociación exitosa del túnel IKE.
- Cifrado correcto del tráfico mediante IPSec.
- Comunicación exitosa entre los extremos de la VPN.
