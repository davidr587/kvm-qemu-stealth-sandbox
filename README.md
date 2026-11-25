# 🛡️ KVM/QEMU Ultimate Stealth Sandbox

![Pafish Status](https://img.shields.io/badge/Pafish-99%25_Bypassed-success)
![Hypervisor](https://img.shields.io/badge/Hypervisor-KVM%2FQEMU-blue)
![OS](https://img.shields.io/badge/OS-Windows_11-blueviolet)

## 📝 Descripción del Proyecto
Este proyecto documenta la configuración de una máquina virtual Windows 11 en KVM/QEMU altamente ofuscada y fortificada para el análisis dinámico de malware. El objetivo principal es evadir las técnicas anti-análisis y anti-VM utilizadas por el malware moderno, logrando superar casi la totalidad de las comprobaciones de la herramienta **Pafish**.

## 💻 Entorno y Hardware
El comportamiento de las latencias de virtualización depende en gran medida del hardware físico. Este laboratorio fue desarrollado y probado en:
* **Host OS:** Linux (Ubuntu/Debian-based)
* **Hardware:** HP Envy Laptop 17-cr1xxx (Intel Core i7-13700H)
* **Hypervisor:** KVM con QEMU y Libvirt
* **Guest OS:** Windows 11 (UEFI/OVMF)

## 🔧 Técnicas de Evasión Implementadas

Se ha modificado la configuración XML del dominio de Libvirt y el registro de Windows para mitigar las firmas clásicas de virtualización:

### 1. CPU & Hypervisor Masking
* Ocultación del flag del hipervisor (`<hidden state='on'/>`) y desactivación del bit `hypervisor` en CPUID.
* Enmascaramiento del `vendor_id` de Hyper-V como `GenuineIntel`.
* Habilitación de `invtsc` para mantener un contador de marcas de tiempo invariante.

### 2. SMBIOS & Hardware Spoofing
* Sobrescritura de los metadatos de la placa base y BIOS para reflejar hardware OEM real (HP Envy, American Megatrends Inc. F.22).
* Falsificación del número de serie del disco duro (`ST1000LM048`) y uso de interfaz de red `e1000e` (Intel) para evitar identificadores de QEMU/VirtIO.

### 3. Clock & Timer Tuning
* Desactivación de `hypervclock` y ajuste de temporizadores (`rtc`, `pit`, `hpet`) para reducir firmas de virtualización en el manejo del reloj.

### 4. Registry Clearing (Guest OS)
* Modificación de las claves `SystemBiosVersion` e `Identifier` (SCSI) en el Registro de Windows mediante el script incluido en `scripts/stealth_registry.bat`.

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
[*] Looking for C:\WINDOWS\system32\drivers\vmmouse.sys
