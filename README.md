# 🛡️ KVM/QEMU Ultimate Stealth Sandbox

![Pafish Status](https://img.shields.io/badge/Pafish-99%25_Bypassed-success)
![Hypervisor](https://img.shields.io/badge/Hypervisor-KVM%2FQEMU-blue)
![OS](https://img.shields.io/badge/OS-Windows_11-blueviolet)

## 📝 Descripción del Proyecto
Documentación para el despliegue de una máquina virtual con Windows 11 en KVM/QEMU optimizada para análisis dinámico de malware. El objetivo es evadir protecciones anti-VM y superar la gran mayoría de las comprobaciones de **Pafish**.

## 💻 Entorno y Hardware
- **Host OS:** Linux (Ubuntu/Debian-based)
- **Hardware:** HP Envy Laptop 17-cr1xxx (Intel Core i7-13700H)
- **Hypervisor:** KVM / QEMU / Libvirt
- **Guest OS:** Windows 11 (UEFI/OVMF)

## 🔧 Técnicas de Evasión Implementadas (XML de Libvirt)
La configuración del dominio (`win11_stealth.xml`) incorpora:
1. **CPU & Hypervisor Masking:** Ocultación del flag de hipervisor, enmascaramiento del `vendor_id` como `GenuineIntel` y activación de `invtsc`.
2. **SMBIOS & Hardware Spoofing:** Emulación de BIOS/placa base OEM (HP Envy), suplantación de número de serie de disco (`ST1000LM048`), uso de controlador de red `e1000e` y OUI de MAC perteneciente a HP.
3. **Clock & Timer Tuning:** Desactivación de "hypervclock" y ajuste de temporizadores (`rtc`, `pit`, `hpet`).

## 📊 Resultados de Pafish
El entorno elude las detecciones de depuradores, Sandboxie, Wine, VirtualBox, VMware, QEMU y Bochs, así como las comprobaciones de *uptime* tras 30 minutos de actividad.

### ⚠️ Limitación Físico-Arquitectónica (RDTSC)
La única verificación que suele registrarse como detectada ("traced!") es la diferencia entre contadores de tiempo de la CPU (`rdtsc`) debido a la gestión de energía y núcleos híbridos en procesadores móviles como el Intel Core i7-13700H.

## 🚀 Instalación y Uso
1. Configura el dominio en Libvirt:
   ```bash
   virsh define win11_stealth.xml
   virsh start win11-hardened
   ```
2. Dejar la máquina encendida al menos 30 minutos antes de realizar pruebas para sortear validaciones de `GetTickCount()`.

---
*Disclaimer: Creado estrictamente con fines académicos y de investigación en ciberseguridad.*
