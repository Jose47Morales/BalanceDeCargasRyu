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

Obtendrás algo asi:

![Uploading image.png…]()

### **Paso 3:** Instalar Ryu

Instala Ryu usando pip:

```bash
pip3 install ryu
```

Debería verse algo así:

![Uploading image.png…]()
