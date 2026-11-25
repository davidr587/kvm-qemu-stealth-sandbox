# 🛡️ KVM/QEMU Ultimate Stealth Sandbox

![Pafish Status](https://img.shields.io/badge/Pafish-99%25_Bypassed-success)
![Hypervisor](https://img.shields.io/badge/Hypervisor-KVM%2FQEMU-blue)
![OS](https://img.shields.io/badge/OS-Windows_11-blueviolet)

## 📝 Descripción del Proyecto
Este proyecto documenta la configuración de una máquina virtual Windows 11 en KVM/QEMU altamente ofuscada y fortificada para el análisis dinámico de malware. El objetivo principal es evadir las técnicas anti-análisis y anti-VM utilizadas por el malware moderno, logrando superar casi la totalidad de las comprobaciones de la herramienta **Pafish**.

## 💻 Entorno y Hardware
El comportamiento de las latencias de virtualización depende en gran medida del hardware físico. Este laboratorio fue desarrollado y probado en:
- **Host OS:** Linux (Ubuntu/Debian-based)
- **Hardware:** HP Envy Laptop 17-cr1xxx (Intel Core i7-13700H)
- **Hypervisor:** KVM con QEMU y Libvirt
- **Guest OS:** Windows 11 (UEFI/OVMF)

## 🔧 Técnicas de Evasión Implementadas

Se ha modificado la configuración XML del dominio de Libvirt (`win11_stealth.xml`) y el registro de Windows para mitigar las firmas clásicas de virtualización:

### 1. CPU & Hypervisor Masking
- Uso de `host-passthrough` (`<cpu mode='host-passthrough'>`) para exponer el modelo real del procesador físico al invitado y evitar la cadena genérica `QEMU Virtual CPU`.
- Ocultación del flag del hipervisor (`<hidden state='on'/>`) y desactivación del bit `hypervisor` en CPUID.
- Enmascaramiento del `vendor_id` de Hyper-V como `GenuineIntel` y habilitación de `invtsc` para mantener un contador de marcas de tiempo invariante.
- Desactivación del puerto I/O *backdoor* de VMware (`<vmport state='off'/>`) para evitar falsos positivos de detección de VMware en KVM.

### 2. SMBIOS & Hardware Spoofing
- Sobrescritura de los metadatos de la placa base y BIOS para reflejar hardware OEM real (HP Envy, American Megatrends Inc. F.22).
- Uso de bus **SATA** (`bus='sata'`) con número de serie falsificado (`ST1000LM048`) en lugar de VirtIO para evitar drivers paravirtualizados de Red Hat/QEMU en el sistema invitado.
- Eliminación del dispositivo *VirtIO Memory Balloon* (`<memballoon model='none'/>`) para suprimir uno de los indicadores más comunes de KVM en el Administrador de Dispositivos.
- Uso de interfaz de red `e1000e` (Intel) y falsificación de la dirección MAC con un OUI real de Hewlett Packard (`3c:52:82:4e:7a:19`) para evitar el prefijo por defecto de QEMU/KVM (`52:54:00`).

### 3. Clock & Timer Tuning
- Desactivación de `hypervclock` y ajuste de temporizadores (`rtc`, `pit`, `hpet`) para reducir firmas de virtualización en el manejo del reloj.

### 4. Registry Clearing (Guest OS)
- Limpieza manual en el Registro de Windows (`regedit`) de las claves `SystemBiosVersion` y el `Identifier` de los dispositivos SCSI para eliminar referencias residuales a QEMU y Bochs.

## 📊 Resultados de Pafish
El entorno supera las detecciones de depuradores, sandboxes genéricas (incluyendo *uptime* tras 30 minutos de encendido), VirtualBox, VMware, Sandboxie, Wine, QEMU y Bochs:

```text
[-] Sandboxie detection
[*] Using GetModuleHandle(sbiedll.dll) ... OK

[-] Wine detection
[*] Using GetProcAddress(wine_get_unix_file_name) from kernel32.dll ... OK
[*] Reg key (HKCU\SOFTWARE\Wine) ... OK

[-] VirtualBox detection
[*] Scsi port->bus->target id->logical unit id-> 0 identifier ... OK
[*] Reg key (HKLM\HARDWARE\Description\System "SystemBiosVersion") ... OK
[*] Reg key (HKLM\SOFTWARE\Oracle\VirtualBox Guest Additions) ... OK
[*] Reg key (HKLM\HARDWARE\Description\System "VideoBiosVersion") ... OK
[*] Reg key (HKLM\HARDWARE\ACPI\DSDT\VBOX__) ... OK
[*] Reg key (HKLM\HARDWARE\ACPI\FADT\VBOX__) ... OK
[*] Reg key (HKLM\HARDWARE\ACPI\RSDT\VBOX__) ... OK
[*] Reg key (HKLM\SYSTEM\ControlSet001\Services\VBox*) ... OK
[*] Reg key (HKLM\HARDWARE\DESCRIPTION\System "SystemBiosDate") ... OK
[*] Driver files in C:\WINDOWS\system32\drivers\VBox* ... OK
[*] Additional system files ... OK
[*] Looking for a MAC address starting with 08:00:27 ... OK
[*] Looking for pseudo devices ... OK
[*] Looking for VBoxTray windows ... OK
[*] Looking for VBox network share ... OK
[*] Looking for VBox processes (vboxservice.exe, vboxtray.exe) ... OK
[*] Looking for VBox devices using WMI ... OK

[-] VMware detection
[*] Scsi port 0,1,2 ->bus->target id->logical unit id-> 0 identifier ... OK
[*] Reg key (HKLM\SOFTWARE\VMware, Inc.\VMware Tools) ... OK
[*] Looking for C:\WINDOWS\system32\drivers\vmmouse.sys ... OK
[*] Looking for C:\WINDOWS\system32\drivers\vmhgfs.sys ... OK
[*] Looking for a MAC address starting with 00:05:69, 00:0C:29, 00:1C:14 or 00:50:56 ... OK
[*] Looking for network adapter name ... OK
[*] Looking for pseudo devices ... OK
[*] Looking for VMware serial number ... OK

[-] Qemu detection
[*] Scsi port->bus->target id->logical unit id-> 0 identifier ... OK
[*] Reg key (HKLM\HARDWARE\Description\System "SystemBiosVersion") ... OK
[*] cpuid CPU brand string 'QEMU Virtual CPU' ... OK

[-] Bochs detection
[*] Reg key (HKLM\HARDWARE\Description\System "SystemBiosVersion") ... OK
[*] cpuid AMD wrong value for processor name ... OK
[*] cpuid Intel wrong value for processor name ... OK
```

### ⚠️ Limitación  (RDTSC)
La única comprobación que permanece en estado `traced!` es:
`Checking the difference between CPU timestamp counters (rdtsc) forcing VM exit`

**Análisis del hallazgo:** En procesadores de arquitectura móvil/híbrida (Intel Core i7-13700H con P-cores y E-cores), la gestión dinámica de energía y los estados C del procesador introducen una latencia de nanosegundos cuando la CPU ejecuta una salida forzada hacia el hipervisor (VM Exit). Salvo en hardware de escritorio dedicado con frecuencias bloqueadas, se trata de una limitación física del silicio anfitrión y no de la configuración de software.

## 🚀 Instalación y Uso

1. **Descargar y adaptar el XML:**
   Descarga `win11_stealth.xml` y ajustar los siguientes campos a tu entorno antes de importarlo:
   - **Disco virtual:** Modificar `<source file='/var/lib/libvirt/images/win11.qcow2'/>` con la ruta real de tu imagen `.qcow2`.
   - **Nombre de la VM:** Por defecto el dominio se llama `win11-hardened` (`<name>win11-hardened</name>`). Si lo cambias, usa ese mismo nombre en el comando `virsh start`.
   - **Rutas UEFI (OVMF):** Verificar que las rutas de `<loader>` y `<nvram>` coincidan con las de tu distribución Linux (las actuales corresponden a Ubuntu/Debian).

2. **Definir e iniciar la VM en Libvirt:**
   ```bash
   virsh define win11_stealth.xml
   virsh start win11-hardened
   ```

3. **Tiempo de actividad (Uptime):**
   Mantén la máquina virtual encendida durante al menos **30 minutos** antes de ejecutar análisis para superar las comprobaciones de `GetTickCount()`.


*Disclaimer: Este entorno fue creado estrictamente con fines académicos y de investigación en ciberseguridad.*
