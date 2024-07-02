# Balanceador de carga en redes WiFi con Ryu y Mininet 

## Introducción 

Este proyecto muestra como implementar un balanceador de carga en redes WiFi utilizando el controlador Ryu y Mininet. Esta guía te ayudará con el proceso de instalación y 
configuración de Ryu y Mininet, así como la implementación de un script para el balanceo de carga. 

## Requisitos

- **Sistema operativo:** Ubuntu 18.04+ desktop o Debian 10+
- **Python 3.6+**
- **Ryu SDN Framework**
- **Mininet**

## Instalación

### **Paso 1:** Actualizar el sistema

Primero, asegurate de que tu sistema esté actualizado, para ello abre una terminal en tu sistema y utiliza los siguientes comandos:

```bash
sudo apt-get update
sudo apt-get upgrade
```

Deberá verse como esto:

![image](https://github.com/Jose47Morales/BalanceDeCargasRyu/assets/149639682/d00f571c-3614-40d4-9037-bb727a28ea49)

![image](https://github.com/Jose47Morales/BalanceDeCargasRyu/assets/149639682/9054d02f-4bb4-4531-b690-4c7df1569ff3)

### **Paso 2:** Instalar dependencias

Instala dependencias necesarias, incluyendo Python y pip:

```bash
sudo apt-get install python3-pip git
```

### **Paso 3:** Instalar Ryu

Instala Ryu usando pip:

```bash
pip3 install ryu
```

Debería verse algo así:

![image](https://github.com/Jose47Morales/BalanceDeCargasRyu/assets/149639682/43ae81e7-3a4e-4fe1-a7b1-605492c1c569)

### **Paso 4:** Instalar Mininet

Clona el repositorio de Mininet y ejecuta el script de instalación:

```bash
git clone https://github.com/mininet/mininet.git
cd mininet
sudo util/install.sh -a
```

Obtendrás algo así:

![image](https://github.com/Jose47Morales/BalanceDeCargasRyu/assets/149639682/570b90f0-1b09-428e-a935-a818ce32fb77)

![image](https://github.com/Jose47Morales/BalanceDeCargasRyu/assets/149639682/91080d7a-2546-45f3-b223-975d5ae08f31)

## Configuración de Mininet

Crea un script de Mininet para configurar la topología de la red con puntos de acceso y switches.

```bash
# archivo: wifi_topology.py

from mininet.net import Mininet
from mininet.node import Controller, OVSKernelSwitch, RemoteController
from mininet.cli import CLI
from mininet.log import setLogLevel, info

def topology():
    net = Mininet(controller=RemoteController, switch=OVSKernelSwitch)

    info('*** Adding controller\n')
    net.addController('c0')

    info('*** Adding switches\n')
    s1 = net.addSwitch('s1')

    info('*** Adding hosts\n')
    h1 = net.addHost('h1', ip='10.0.0.1')
    h2 = net.addHost('h2', ip='10.0.0.2')

    info('*** Adding access points\n')
    ap1 = net.addSwitch('ap1')
    ap2 = net.addSwitch('ap2')

    info('*** Creating links\n')
    net.addLink(h1, s1)
    net.addLink(h2, s1)
    net.addLink(s1, ap1)
    net.addLink(s1, ap2)

    info('*** Starting network\n')
    net.start()

    CLI(net)

    info('*** Stopping network\n')
    net.stop()

if __name__ == '__main__':
    setLogLevel('info')
    topology()
```
