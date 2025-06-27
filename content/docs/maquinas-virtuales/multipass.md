---
title: "Multipass - Guía Completa"
weight: 1
bookToc: true
---

# Multipass: Máquinas Virtuales Ubuntu Simplificadas

Multipass es una herramienta desarrollada por Canonical que permite crear y gestionar máquinas virtuales Ubuntu de forma rápida y sencilla. Es perfecta para desarrollo, testing y experimentación.

## 🚀 Instalación

### Instalación básica

Descarga Multipass desde la [página oficial](https://multipass.run/docs/tutorial) y sigue las instrucciones para tu sistema operativo.

### Verificar la instalación

```bash
multipass version
```

## 📋 Comandos básicos

### Crear una máquina virtual

```bash
# VM básica con configuración por defecto
multipass launch --name labs

# VM con recursos específicos
multipass launch --name labs --memory 4G --cpus 2 --disk 20G
```

### Crear VM con cloud-init

```bash
multipass launch --name labs --cloud-init cloud-init.yaml --memory 4G --cpus 2
```

## ⚙️ Configuración con Cloud-init

### Archivo `cloud-init.yaml`

```yaml
#cloud-config

# Actualiza los paquetes del sistema
package_update: true
package_upgrade: true

# Lista de paquetes a instalar
packages:
  - git
  - docker.io
  - python3
  - python3-pip
  - python3-venv
  - curl
  - unzip
  - awscli
  - openssh-client

# Comandos a ejecutar después de la instalación de paquetes
runcmd:
  - |
      # Instalar kubectl
      curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
      chmod +x kubectl
      mv kubectl /usr/local/bin/
  - |
      # Instalar minikube
      curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
      chmod +x minikube
      mv minikube /usr/local/bin/
  - |
      # Instalar tfenv
      curl -Lo tfenv.zip https://github.com/tfutils/tfenv/archive/refs/heads/master.zip
      unzip tfenv.zip -d /usr/local/
      mv /usr/local/tfenv-master /usr/local/tfenv
      echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bashrc
      ln -s /usr/local/tfenv/bin/tfenv /usr/local/bin/tfenv
  - |
      # Instalar Terraform usando tfenv
      tfenv install latest
      tfenv use latest
  - |
      # Configurar Docker
      usermod -aG docker $USER
      systemctl enable docker
      systemctl start docker
  - |
      # Generar una clave SSH
      mkdir -p ~/.ssh
      ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ''
      eval "$(ssh-agent -s)"
      ssh-add ~/.ssh/id_rsa

# Mensaje final cuando todo está completo
final_message: "La instancia de Multipass está configurada y lista para usar."
```

## 🔧 Gestión de máquinas virtuales

### Listar VMs

```bash
multipass list
```

### Acceder a una VM

```bash
multipass shell labs
```

### Información de una VM

```bash
multipass info labs
```

### Detener una VM

```bash
multipass stop labs
```

### Iniciar una VM

```bash
multipass start labs
```

### Eliminar una VM

```bash
multipass delete labs
multipass purge
```

## ⚙️ Modificar configuración de VMs

Para modificar los recursos de una VM existente, primero debes detenerla:

```bash
# Detener la VM
multipass stop handsome-ling

# Modificar recursos
multipass set local.handsome-ling.cpus=4
multipass set local.handsome-ling.disk=60G
multipass set local.handsome-ling.memory=7G

# Iniciar la VM
multipass start handsome-ling
```

> **Nota**: `handsome-ling` es el nombre de la instancia. Reemplaza con el nombre de tu VM.

## 📁 Compartir archivos

### Montar directorios

```bash
# Montar directorio del host en la VM
multipass mount /ruta/local labs:/ruta/en/vm

# Desmontar
multipass umount labs:/ruta/en/vm
```

### Transferir archivos

```bash
# Copiar archivo al VM
multipass transfer archivo.txt labs:~/

# Copiar archivo desde VM
multipass transfer labs:~/archivo.txt ./
```

## 📸 Snapshots

### Crear snapshot

```bash
multipass snapshot create labs snapshot-inicial
```

### Listar snapshots

```bash
multipass snapshot list labs
```

### Restaurar snapshot

```bash
multipass snapshot restore labs snapshot-inicial
```

### Eliminar snapshot

```bash
multipass snapshot delete labs snapshot-inicial
```

## 🌐 Configuración de red

### Obtener IP de la VM

```bash
multipass info labs | grep IPv4
```

### Configurar port forwarding

```bash
# Ejemplo para acceder a un servicio web en puerto 8080
multipass exec labs -- sudo ufw allow 8080
```

## 🔍 Solución de problemas

### Verificar el servicio

```bash
# En Linux/macOS
sudo systemctl status multipass

# En Windows (PowerShell como administrador)
Get-Service -Name multipass
```

### Reiniciar el servicio

```bash
# En Linux/macOS
sudo systemctl restart multipass

# En Windows (PowerShell como administrador)
Restart-Service -Name multipass
```

### Logs de Multipass

```bash
# Ver logs
multipass logs labs

# Logs del sistema
journalctl -u multipass
```

## 🎯 Casos de uso prácticos

### Entorno de desarrollo web

```yaml
#cloud-config
packages:
  - nodejs
  - npm
  - nginx
  - git

runcmd:
  - npm install -g yarn
  - systemctl enable nginx
  - systemctl start nginx
```

### Entorno Docker

```yaml
#cloud-config
packages:
  - docker.io
  - docker-compose

runcmd:
  - usermod -aG docker ubuntu
  - systemctl enable docker
  - systemctl start docker
```

### Entorno Python

```yaml
#cloud-config
packages:
  - python3
  - python3-pip
  - python3-venv
  - python3-dev

runcmd:
  - pip3 install virtualenv
  - pip3 install jupyter
```

## 📚 Recursos adicionales

- [Documentación oficial](https://multipass.run/docs)
- [Modificar instancias](https://multipass.run/docs/modify-an-instance)
- [GUI Client](https://multipass.run/docs/multipass-gui-client)
- [Tutorial oficial](https://multipass.run/docs/tutorial)

¡Con Multipass puedes crear entornos de desarrollo Ubuntu limpios y configurados en minutos!
