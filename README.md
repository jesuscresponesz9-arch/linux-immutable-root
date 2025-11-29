# 🛡️ **BASTIÓN INMUTABLE** — **Linux Immutable Root Configuration**

## 🏛️ **Arquitectura de Resiliencia con OverlayFS para Servidores Críticos**

##   

## 📖 **Descripción General**

Este proyecto implementa un sistema Linux con el **Root Filesystem (/) configurado como solo lectura**. Utilizando **OverlayFS**, cualquier modificación o intrusión en el sistema operativo es temporal y se pierde al reiniciar.

Este diseño es fundamental para la **defensa de la infraestructura** y la prevención de *configuration drift*, siendo la solución ideal para:

  * **Servidores Bastión (Jump Hosts)**
  * **Kioskos y Puestos de Control**
  * **Infraestructuras de Alta Seguridad**

-----

## 🎯 **Principios Fundamentales**

| Icono | Característica | Beneficio Arquitectónico |
| :--- | :--- | :--- |
| ✅ | **Inmutabilidad de Arranque** | El sistema operativo base en el disco duro nunca se modifica, garantizando la **integridad de la fuente**. |
| ✅ | **Recuperación Zero-Touch** | Ante un ataque o compromiso, la restauración es automática y requiere únicamente un reinicio (`Zero-touch restoration`). |
| ✅ | **Mantenimiento Controlado** | Los cambios permanentes solo son posibles mediante un proceso de *whitelist* y elevación de privilegios (`overlayroot-chroot`). |

-----

## 🏗️ **Arquitectura del Sistema: Separación de Capas**

El diseño se basa en **OverlayFS** para superponer un sistema de archivos temporal de escritura sobre el *root* inmutable del disco.

```ascii
[ CAPA DE ESCRITURA (RAM/tmpfs) ]  ← Cambios Temporales (Volátiles) ⚡
                        |
[ VISTA UNIFICADA (OverlayFS) ]    ← Lo que ve el Usuario/Proceso 👁️
                        |
[ CAPA DE LECTURA (Disco Duro) ]   ← Sistema Base Inmutable (Persistente) 💾
```

1.  **Capa Inferior (Disco Duro):** El sistema operativo base. Es de **solo lectura** y define el estado "sano".
2.  **Capa Superior (RAM/tmpfs):** Un sistema de archivos en memoria (RAM) que registra todas las escrituras y cambios. Es **volátil**.
3.  **OverlayFS:** Presenta la vista unificada a los procesos. Al reiniciar, la Capa Superior (RAM) se borra, y el sistema vuelve a la Capa Inferior (Disco).

-----

## 🚀 **Instalación y Configuración**

### **Prerrequisitos**

  * **Distribución:** Ubuntu Server 20.04/22.04.
  * **Acceso:** `sudo`/`root`.

### **Procedimiento de Inmutabilidad**

```bash
# 1. Instalar el paquete overlayroot
sudo apt install overlayroot -y

# 2. Configurar la inmutabilidad usando tmpfs (RAM)
# La configuración 'tmpfs' garantiza que los cambios son volátiles.
echo 'overlayroot="tmpfs"' | sudo tee /etc/overlayroot.conf

# 3. Aplicar y reiniciar el sistema
sudo reboot
```

### **Verificación de Modo Activo**

```bash
# Verificar la montura OverlayFS
sudo mount | grep overlay

# Salida esperada que confirma el modo overlay (rw = write layer)
# overlayroot on / type overlay (rw,relatime,lowerdir=/media/root-ro,...)
```

-----

## 🧪 **Demostración de Resiliencia**

### 🔴 **Estado Comprometido (Pre-Reinicio)**

Se simula un escenario de ataque o manipulación de la configuración:

```bash
# Simulación de un ataque
sudo rm -rf /etc/nginx
touch /home/usuario/hackeado.txt
echo "SISTEMA COMPROMETIDO" | sudo tee /etc/version_sistema

# Verificar que los daños son visibles
ls /etc/nginx                         # El directorio NO existe
sudo systemctl start nginx            # El servicio FALLA

# [ Evidencia Visual: Captura de Sistema Comprometido ]
```

### 🟢 **Estado Restaurado (Post-Reinicio)**

Tras un simple reinicio, el sistema vuelve a su estado inmutable original, **eliminando toda la actividad maliciosa**:

```bash
# 1. Ejecutar el reinicio
sudo reboot

# 2. Verificar la recuperación automática
ls /etc/nginx/                        # Directorio RESTAURADO
sudo systemctl status nginx           # Servicio ACTIVO
ls /home/usuario/hackeado.txt         # Archivo ELIMINADO

# [ Evidencia Visual: Captura de Sistema Restaurado ]
```

-----

## 🛠️ **Mantenimiento y Actualizaciones (Proceso Controlado)**

Para realizar un cambio permanente (ej. una actualización de seguridad), se debe acceder a la capa base de solo lectura usando el comando **`overlayroot-chroot`**:

```bash
# 1. Acceder al sistema base de LECTURA/ESCRITURA (RW)
sudo overlayroot-chroot

# 2. Realizar los cambios persistentes (Ej. actualización)
echo "Versión 1.1 - Actualizado permanentemente" > /etc/version_sistema
apt update && apt upgrade -y

# 3. Salir y reiniciar (para aplicar los cambios de forma inmutable)
exit
sudo reboot
```

-----

## 📊 **Casos de Uso Detallados**

| Caso de Uso | Enfoque de Seguridad | Beneficio Clave |
| :--- | :--- | :--- |
| **Servidor Bastión** | Protección contra modificaciones no autorizadas (Técnica: **Anti-Intrusión**). | **Recuperación automática** post-intrusión. |
| **Laboratorios/Educación** | Restauración a estado conocido (Técnica: **Limpieza de Sesión**). | Prevención de modificaciones accidentales o persistentes. |
| **Kioskos/Públicos** | Resistencia a la manipulación de usuario final (Técnica: **Hardening de Interfaz**). | **Mantenimiento simplificado** y alta disponibilidad. |

-----

## 📝 **Pruebas Realizadas**

| Prueba Simulada | Objetivo de Seguridad | Resultado |
| :--- | :--- | :--- |
| Eliminación de `/etc/nginx` | Integridad de Configuración | ✅ **Restaurado** |
| Creación de archivos temporales | Anti-Malware / Anti-Persistencia | ✅ **Eliminados** |
| Modificación de archivos binarios | Anti-Rootkit | ✅ **Revertida** |
| Cambios vía `overlayroot-chroot` | Persistencia Controlada | ✅ **Persistentes** |

-----

## 📄 **Licencia y Contribuciones**

### **Contribuciones**

¡Contribuciones son bienvenidas\! Por favor, siga el proceso estándar de Pull Request: *Fork*, crear una rama para su *feature*, *commit* y abrir un *Pull Request*.

### **Licencia**

Este proyecto está bajo la Licencia **MIT** — ver el archivo [LICENSE](https://www.google.com/search?q=LICENSE) para detalles.

-----
