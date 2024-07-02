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

## Implementación de balanceador de carga en Ryu

Crea una aplicación de Ryu para gestionar el balanceo de carga:

```bash
# archivo: load_balancer.py

from ryu.base import app_manager
from ryu.controller import ofp_event
from ryu.controller.handler import CONFIG_DISPATCHER, MAIN_DISPATCHER
from ryu.controller.handler import set_ev_cls
from ryu.ofproto import ofproto_v1_3
from ryu.lib.packet import packet
from ryu.lib.packet import ethernet

class LoadBalancer(app_manager.RyuApp):
    OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]

    def __init__(self, *args, **kwargs):
        super(LoadBalancer, self).__init__(*args, **kwargs)
        self.mac_to_port = {}

    @set_ev_cls(ofp_event.EventOFPSwitchFeatures, CONFIG_DISPATCHER)
    def switch_features_handler(self, ev):
        datapath = ev.msg.datapath
        ofproto = datapath.ofproto
        parser = datapath.ofproto_parser

        match = parser.OFPMatch()
        actions = [parser.OFPActionOutput(ofproto.OFPP_CONTROLLER, ofproto.OFPCML_NO_BUFFER)]
        self.add_flow(datapath, 0, match, actions)

    def add_flow(self, datapath, priority, match, actions):
        ofproto = datapath.ofproto
        parser = datapath.ofproto_parser
        inst = [parser.OFPInstructionActions(ofproto.OFPIT_APPLY_ACTIONS, actions)]
        mod = parser.OFPFlowMod(datapath=datapath, priority=priority, match=match, instructions=inst)
        datapath.send_msg(mod)

    @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
    def _packet_in_handler(self, ev):
        msg = ev.msg
        datapath = msg.datapath
        ofproto = datapath.ofproto
        parser = datapath.ofproto_parser
        in_port = msg.match['in_port']

        pkt = packet.Packet(msg.data)
        eth = pkt.get_protocols(ethernet.ethernet)[0]

        dst = eth.dst
        src = eth.src

        dpid = datapath.id
        self.mac_to_port.setdefault(dpid, {})

        self.logger.info("packet in %s %s %s %s", dpid, src, dst, in_port)

        self.mac_to_port[dpid][src] = in_port

        if dst in self.mac_to_port[dpid]:
            out_port = self.mac_to_port[dpid][dst]
        else:
            out_port = ofproto.OFPP_FLOOD

        actions = [parser.OFPActionOutput(out_port)]

        if out_port != ofproto.OFPP_FLOOD:
            match = parser.OFPMatch(in_port=in_port, eth_dst=dst, eth_src=src)
            self.add_flow(datapath, 1, match, actions)

        data = None
        if msg.buffer_id == ofproto.OFP_NO_BUFFER:
            data = msg.data

        out = parser.OFPPacketOut(datapath=datapath, buffer_id=msg.buffer_id, in_port=in_port, actions=actions, data=data)
        datapath.send_msg(out)
```
