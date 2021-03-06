---
title: El apagado de máquinas virtuales de Azure está atascado en el reinicio, apagado o detención de servicios | Microsoft Docs
description: Este artículo le ayuda a solucionar errores de servicio en las máquinas virtuales Windows en Azure.
services: virtual-machines-windows
documentationCenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 12/19/2019
ms.author: tibasham
ms.openlocfilehash: 5d6396efc9ab25baa0d32e7c33c7715863516249
ms.sourcegitcommit: f255f869c1dc451fd71e0cab340af629a1b5fb6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/16/2020
ms.locfileid: "77371360"
---
# <a name="azure-windows-vm-shutdown-is-stuck-on-restarting-shutting-down-or-stopping-services"></a>El apagado de máquinas virtuales Windows en Azure está atascado en "Reiniciando", "Apagando" o "Deteniendo servicios".

En este artículo se proporcionan los pasos necesarios para resolver los problemas de los mensajes "Reiniciando", "Apagando" o "Deteniendo servicios" que pueden aparecer al reiniciar una máquina virtual Windows en Microsoft Azure.

## <a name="symptoms"></a>Síntomas

Cuando use los [diagnósticos de arranque](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/boot-diagnostics) para ver la captura de pantalla de la máquina virtual, es posible que vea que la captura de pantalla muestra el mensaje "Reiniciando", "Apagando" o "Deteniendo servicios".

![Pantallas Reiniciando, Apagando o Deteniendo servicios](./media/boot-error-troubleshooting-windows/restart-shut-down-stop-service.png)
 
## <a name="cause"></a>Causa

Windows usa el proceso de apagado tanto para realizar operaciones de mantenimiento del sistema como para procesar cambios como actualizaciones, roles y características. No es aconsejable interrumpir este proceso crítico hasta que se complete. En función del número de actualizaciones o cambios, y del tamaño de la máquina virtual, el proceso puede durar mucho tiempo. Si el proceso se detiene, es posible que el sistema operativo resulte dañado. El proceso solo se debe interrumpir si tarda demasiado tiempo en finalizar.

## <a name="solution"></a>Solución

### <a name="collect-a-process-memory-dump"></a>Recopilar un volcado de memoria de proceso

1. Descargue la [herramienta Procdump](http://download.sysinternals.com/files/Procdump.zip) en un disco de datos nuevo o existente, que esté conectado a una máquina virtual en funcionamiento de la misma región.

2. Desconecte el disco que contiene los archivos necesarios de la máquina virtual en funcionamiento y conéctelo a la máquina virtual rota. Vamos a llamar a este disco **Disco de utilidad**.

Use la [Consola serie](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/serial-console-windows) para completar los pasos siguientes:

1. Abra un PowerShell administrativo y compruebe el servicio que se bloquea al detenerse.

   ``
   Get-Service | Where-Object {$_.Status -eq "STOP_PENDING"}
   ``

2. En un CMD administrativo, obtenga el PID del servicio bloqueado.

   ``
   tasklist /svc | findstr /i <STOPING SERVICE>
   ``

3. Obtenga un ejemplo de volcado de memoria de proceso bloqueado <STOPPING SERVICE>.

   ``
   procdump.exe -s 5 -n 3 -ma <PID>
   ``

4. Ahora, elimine el proceso bloqueado para desbloquear el proceso de apagado.

   ``
   taskkill /PID <PID> /t /f
   ``

Una vez que el sistema operativo se vuelva a iniciar, si arranca con normalidad, asegúrese de que la coherencia del sistema operativo sea correcta. Si se detectan daños en el disco, ejecute el siguiente comando hasta que desaparezcan:

``
dism /online /cleanup-image /restorehealth
``

Si no puede recopilar un volcado de memoria de proceso, o si este problema es recursivo y necesita un análisis de la causa raíz, pase a recopilar un volcado de memoria del sistema operativo, el procedimiento para abrir una solicitud de soporte técnico.

### <a name="collect-an-os-memory-dump"></a>Recopilación de un volcado de memoria del sistema operativo

Si el problema no se soluciona después de esperar a que se hayan procesado los cambios, deberá recopilar un archivo de volcado de memoria y ponerse en contacto con el soporte técnico. Para recopilar el archivo de volcado de memoria, siga estos pasos:

**Conecte el disco del sistema operativo a una máquina virtual de recuperación**

1. Tome una instantánea del disco del sistema operativo de la máquina virtual afectada como copia de seguridad. Para obtener más información, consulte [Instantánea de un disco](https://docs.microsoft.com/azure/virtual-machines/windows/snapshot-copy-managed-disk).

2. [Conecte el disco del sistema operativo a una máquina virtual de recuperación](https://docs.microsoft.com/azure/virtual-machines/windows/troubleshoot-recovery-disks-portal).

3. Escritorio remoto a la máquina virtual de recuperación.

4. Si el disco del sistema operativo está cifrado, debe desactivar el cifrado antes de pasar al siguiente paso. Para más información, consulte [Descifrado del disco de sistema cifrado en la máquina virtual que no arranca](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/troubleshoot-bitlocker-boot-error#solution).

**Busque el archivo de volcado de memoria y envíe una incidencia de soporte técnico**

1. En la máquina virtual de recuperación, vaya a la carpeta de Windows en el disco del sistema operativo conectado. Si la letra de unidad que se asigna al disco del sistema operativo conectado es F, deberá ir a F:\Windows.

2. Busque el archivo memory.dmp y, a continuación, [envíe una incidencia de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) con el archivo de volcado de memoria.

Si no encuentra el archivo de volcado de memoria, vaya el paso siguiente para habilitar el registro de volcado de memoria y Serial Console.

**Habilite el registro del volcado de memoria y la consola serie**

Para habilitar el registro de volcado de memoria y Serial Console, ejecute el siguiente script.

1. Abra la sesión de símbolo del sistema con privilegios elevados (Ejecutar como administrador).

2. Ejecute el siguiente script:

   En este script, se supone que la letra de unidad que se asigna al disco del sistema operativo conectado es F. Reemplácela por el valor adecuado en la máquina virtual.

   ```
   reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv
   
   REM Enable Serial Console
   bcdedit /store F:\boot\bcd /set {bootmgr} displaybootmenu yes
   bcdedit /store F:\boot\bcd /set {bootmgr} timeout 5
   bcdedit /store F:\boot\bcd /set {bootmgr} bootems yes
   bcdedit /store F:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
   bcdedit /store F:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
   
   REM Suggested configuration to enable OS Dump
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump / t REG_DWORD /d 1 /f
   
   reg unload HKLM\BROKENSYSTEM
   ```

3. Compruebe que hay suficiente espacio en el disco para asignar tanta memoria como la RAM, que depende del tamaño que se seleccione para esta máquina virtual.

4. Si no hay suficiente espacio o la máquina virtual es grande (serie G, GS o E), puede cambiar la ubicación en la que se creará este archivo y hacer referencia a cualquier otro disco de datos conectado a la máquina virtual. Para cambiar la ubicación, debe cambiar la siguiente clave:

   ```
   reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f
   
   reg unload HKLM\BROKENSYSTEM
   ```

5. [Desconecte el disco del sistema operativo y, después, vuelva a conectarlo a la máquina virtual afectada](https://docs.microsoft.com/azure/virtual-machines/windows/troubleshoot-recovery-disks-portal).

6. Inicie la máquina virtual y acceda a la consola serie.

7. Seleccione Enviar interrupción no enmascarable (NMI) para desencadenar el volcado de memoria.

   ![Enviar interrupción no enmascarable](./media/boot-error-troubleshooting-windows/send-nonmaskable-interrupt.png)

8. Vuelva a conectar el disco del sistema operativo a una máquina virtual de recuperación y recopile el archivo de volcado.

## <a name="contact-microsoft-support"></a>Consultar al soporte técnico de Microsoft

Después de recopilar el archivo de volcado de memoria, póngase en contacto con el servicio de soporte técnico de Microsoft para determinar la causa principal.
