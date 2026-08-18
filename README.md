# HW-03 - Configuración de Red en Máquinas Virtuales

En esta actividad se utilizaron máquinas virtuales para conectar diferentes modos de red, configurando una VM en modo **Bridge** bajo tres escenarios distintos: IP por DHCP, IP manual dentro de la subred del hipervisor, e IP manual fuera de la subred del hipervisor.

- **Hipervisor:** VirtualBox
- **Sistema operativo de la VM:** Ubuntu Server 26.04 LTS
- **Hostname configurado:** `santi`
- **Interfaz de red de la VM:** `enp0s3`
- **Modo de red:** Adaptador puente (Bridge), conectado al adaptador Wi-Fi físico del hipervisor (Qualcomm QCA9377)

---

## Subred del hipervisor

Antes de configurar la VM, se identificó la subred del hipervisor mediante `ipconfig` en el host físico.

| Dato | Valor |
|---|---|
| Dirección IPv4 del host | 192.168.1.5 |
| Máscara de subred | 255.255.255.0 |
| Puerta de enlace | 192.168.1.1 |
| **Subred del hipervisor** | **192.168.1.0/24** |

<img width="642" height="155" alt="image" src="https://github.com/user-attachments/assets/101cd343-7351-4d37-ade3-e3391da593c2" />


---

## Escenario 1: Bridge + IP por DHCP

Con la VM configurada en modo Bridge y la interfaz `enp0s3` en modo DHCP (`dhcp4: true`), se obtuvo una IP automáticamente dentro del rango de la subred del hipervisor.

**IP obtenida:** `192.168.1.6/24`

**Comandos ejecutados:**
```bash
ip a
ping -c 4 google.com
```

**Resultado del ping:** 4 paquetes transmitidos, 4 recibidos, 0% packet loss.

<img width="899" height="532" alt="image" src="https://github.com/user-attachments/assets/cea64462-3aa1-4d97-94fd-ab34d0dd425a" />


---

## Escenario 2: Bridge + IP manual dentro de la subred del hipervisor

Se modificó el archivo `/etc/netplan/00-installer-config.yaml` para asignar una IP estática dentro del mismo rango de la subred del hipervisor (`192.168.1.0/24`).

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      dhcp6: false
      addresses:
        - 192.168.1.150/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      match:
        macaddress: 08:00:27:20:d5:26
      set-name: enp0s3
```

**IP asignada:** `192.168.1.150/24`

**Comandos ejecutados:**
```bash
sudo netplan apply
ip a
ping -c 4 google.com
```

**Resultado del ping:** 4 paquetes transmitidos, 4 recibidos, 0% packet loss.

Al estar dentro de la misma subred y con el gateway correcto (`192.168.1.1`), la VM mantiene conectividad total a internet.

<img width="916" height="492" alt="image" src="https://github.com/user-attachments/assets/0f434ac2-15d8-4557-82ec-f0942e7264fe" />


---

## Escenario 3: Bridge + IP manual fuera de la subred del hipervisor

Se modificó nuevamente el archivo netplan, esta vez asignando una IP perteneciente a una red distinta (`10.10.10.0/24`), fuera del rango del hipervisor, manteniendo el mismo gateway real (`192.168.1.1`).

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      dhcp6: false
      addresses:
        - 10.10.10.150/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      match:
        macaddress: 08:00:27:20:d5:26
      set-name: enp0s3
```

**IP asignada:** `10.10.10.150/24`

**Comandos ejecutados:**
```bash
sudo netplan apply
ip a
ping -c 4 google.com
```

**Resultado del ping:** `Temporary failure in name resolution` ❌

**Explicación:**
Al asignar `10.10.10.150/24`, la VM asume que su red local es `10.10.10.0/24`. El gateway real de la red física (`192.168.1.1`) no pertenece a ese segmento, por lo que la tabla de rutas de la VM no puede alcanzarlo. Como resultado, ni siquiera es posible consultar el servidor DNS (8.8.8.8), ya que el tráfico para llegar a él también debía pasar primero por ese gateway inalcanzable. Esto confirma que, para que una interfaz configurada manualmente tenga conectividad, su IP y su gateway deben pertenecer a la misma subred.

<img width="894" height="373" alt="image" src="https://github.com/user-attachments/assets/357ca6ee-f13d-462d-a705-a122225e56ea" />


---

## Conclusiones

- El modo Bridge conecta la VM directamente a la red física del hipervisor, comportándose como un dispositivo más dentro de esa red.
- Tanto con DHCP como con una IP manual dentro de la subred correcta, la VM obtiene conectividad completa a internet.
- Asignar una IP fuera de la subred del gateway rompe el enrutamiento, incluso si el resto de la configuración (DNS, interfaz, cableado virtual) es correcta, ya que el sistema operativo no puede alcanzar un gateway que no pertenece a su propio segmento de red.
