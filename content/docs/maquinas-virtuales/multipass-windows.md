---
title: "Multipass en Windows Home"
weight: 2
bookToc: true
---

# Pasos para usar Multipass en Windows Home con VirtualBox

Windows Home no incluye Hyper-V, por lo que necesitamos usar VirtualBox como backend para Multipass. Esta guía te llevará paso a paso para configurar todo correctamente.

## 1️⃣ Instalar los requisitos previos

### 🔹 **Instalar VirtualBox**

Si no lo tienes instalado:

1. Descarga VirtualBox desde: [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)
2. Instálalo con la configuración por defecto.
3. **Reinicia tu PC** tras la instalación.

### 🔹 **Instalar Multipass**

1. Descarga Multipass desde: [https://multipass.run/download](https://multipass.run/download)
2. Durante la instalación, elige **VirtualBox** como backend si te lo solicita.
3. Completa la instalación y reinicia el sistema.

---

## 2️⃣ Verificar la instalación

Abre **PowerShell** y ejecuta:

```powershell
multipass version
```

Debe mostrar la versión de Multipass instalada.

Para asegurarte de que está usando VirtualBox, ejecuta:

```powershell
multipass get local.driver
```

Si devuelve `hyperv`, debes cambiarlo manualmente.

---

## 3️⃣ Configurar Multipass para usar VirtualBox

Si Multipass está intentando usar **Hyper-V**, configúralo para usar **VirtualBox**:

```powershell
multipass set local.driver=virtualbox
```

Verifica que el cambio se aplicó:

```powershell
multipass get local.driver
```

Debe devolver: `virtualbox`

### ⚠️ **Solución de problemas**

Si el problema persiste, chequear la documentación seteando SO Windows.

**Chequear si `multipass` está corriendo**

```powershell
Get-Service -Name multipass
```

Salida esperada:

```powershell
Status   Name               DisplayName
------   ----               -----------
Running  multipass          Multipass Service
```

**Iniciar servicio de `multipass` si no está corriendo**

```powershell
Start-Service -Name multipass
```

---

## 4️⃣ Crear una máquina virtual

Para crear una VM Ubuntu:

```powershell
multipass launch --name ubuntu-vm
```

Si deseas asignar más recursos:

```powershell
multipass launch --name ubuntu-vm --cpus 2 --mem 2G --disk 10G
```

---

## 5️⃣ Administrar máquinas virtuales

📌 **Listar VMs activas**:

```powershell
multipass list
```

📌 **Acceder a la VM**:

```powershell
multipass shell ubuntu-vm
```

📌 **Detener la VM**:

```powershell
multipass stop ubuntu-vm
```

📌 **Eliminar una VM**:

```powershell
multipass delete ubuntu-vm
multipass purge
```

## 🔧 Comandos útiles adicionales

### Información de la VM

```powershell
multipass info ubuntu-vm
```

### Ejecutar comandos remotos

```powershell
multipass exec ubuntu-vm -- ls -la
multipass exec ubuntu-vm -- sudo apt update
```

### Transferir archivos

```powershell
# Copiar archivo a la VM
multipass transfer archivo.txt ubuntu-vm:~/

# Copiar archivo desde la VM
multipass transfer ubuntu-vm:~/archivo.txt ./
```

### Montar directorios

```powershell
# Montar carpeta de Windows en la VM
multipass mount C:\Users\TuUsuario\Documentos ubuntu-vm:~/documentos

# Desmontar
multipass umount ubuntu-vm:~/documentos
```

## 🎯 Ejemplo práctico: VM para desarrollo

### Crear VM con configuración específica

```powershell
multipass launch --name dev-env --cpus 4 --mem 4G --disk 20G
```

### Configurar la VM para desarrollo

```powershell
# Acceder a la VM
multipass shell dev-env

# Dentro de la VM, instalar herramientas
sudo apt update
sudo apt install -y git nodejs npm python3 python3-pip docker.io
sudo usermod -aG docker $USER
```

### Usar la VM

```powershell
# Montar tu directorio de proyectos
multipass mount C:\Users\TuUsuario\Proyectos dev-env:~/proyectos

# Acceder y trabajar
multipass shell dev-env
cd ~/proyectos
# ... trabajar normalmente
```

## 🚨 Solución de problemas comunes

### Error: "multipass launch failed"

1. Verifica que VirtualBox esté instalado y funcionando
2. Asegúrate de que el servicio de Multipass esté corriendo:
   ```powershell
   Get-Service -Name multipass
   Start-Service -Name multipass
   ```

### Error: "Unable to start VM"

1. Verifica que la virtualización esté habilitada en BIOS/UEFI
2. Cierra VirtualBox si está abierto
3. Reinicia el servicio de Multipass:
   ```powershell
   Restart-Service -Name multipass
   ```

### VM no obtiene IP

1. Verifica la configuración de red en VirtualBox
2. Reinicia la VM:
   ```powershell
   multipass restart ubuntu-vm
   ```

## 📹 Video de ejemplo

Para ver un ejemplo práctico en video: [https://youtu.be/BnH1OdIvN1c](https://youtu.be/BnH1OdIvN1c?si=rRU9LiRDjbyTmM1o)

## 🎉 ¡Listo!

Ahora tienes Multipass funcionando en Windows Home con VirtualBox. Puedes crear máquinas virtuales Ubuntu fácilmente para desarrollo, testing o experimentación.

### Próximos pasos recomendados:

1. Experimenta creando diferentes VMs para diferentes proyectos
2. Aprende a usar cloud-init para automatizar la configuración
3. Explora los snapshots para guardar estados específicos
4. Configura montajes de directorios para un flujo de trabajo eficiente

¡Disfruta desarrollando con entornos Ubuntu limpios y aislados!
