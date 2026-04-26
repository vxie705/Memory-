# Memory-
Rootkit detector based on analyzing inconsistencies between the system handles table and Toolhelp32 process snapshots. Implements Floating Thread Detection in private memory and RWX region scanning, integrating a SIEM telemetry pipeline for remote incident monitoring.

# Advanced Rootkit & Memory Detector

**MEMORY** es una herramienta de monitoreo y detección de amenazas a nivel de kernel y memoria para sistemas Windows. Está diseñada para identificar procesos ocultos, inyecciones de código (shellcode) y comportamientos anómalos que los antivirus convencionales suelen pasar por alto.

> [!WARNING]
> **DISCLAIMER:** Esta herramienta tiene la capacidad de suspender procesos del sistema (Quarantine). Úsela bajo su propio riesgo. El autor no se hace responsable por inestabilidad del sistema o pérdida de datos.

## 🚀 Características Principales

### 1. Detección de Procesos Ocultos (Native Stealth Detection)
Utiliza la API nativa de Windows (`NtQuerySystemInformation`) con la clase `SystemExtendedHandleInformation` para enumerar procesos a través de la tabla de handles del sistema. Esto permite detectar procesos que se han ocultado de la lista oficial de procesos (DKOM - Direct Kernel Object Manipulation).

### 2. Deep Thread Validation (Floating Threads)
Analiza los hilos de ejecución para encontrar **"Floating Threads"**: hilos que comienzan en regiones de memoria privada (`MEM_PRIVATE`) sin un módulo o archivo asociado en el disco. Esta es una técnica clásica de detección de inyección de Shellcode.

### 3. Análisis de Memoria RWX
Escanea el espacio de direcciones de los procesos en busca de regiones con permisos de **Lectura, Escritura y Ejecución (RWX)**, un indicador común de desempaquetadores (unpackers) y malware polimórfico.

### 4. Cuarentena Automática
Implementa un sistema de puntuación de riesgo (Risk Scoring). Si un proceso es marcado como **CRITICAL** (ej. no está firmado, está oculto y tiene hilos inyectados), el motor invoca `NtSuspendProcess` para neutralizar la amenaza instantáneamente.

### 5. SIEM Pipeline
Incluye un pipeline de telemetría que envía eventos en formato JSON a través de UDP (Syslog style), permitiendo la integración con herramientas de monitoreo como Splunk, ELK o servidores Syslog locales.

## 🛠️ Tecnologías Utilizadas
- **Lenguaje:** C++
- **APIs:** Win32 API, NTAPI (ntdll.dll), WinTrust, IP Helper API.
- **Librerías:** Winsock2 para telemetría, Psapi para análisis de módulos.

## 📋 Requisitos y Compilación

Para compilar el proyecto en Visual Studio:
1. Asegúrate de incluir las siguientes librerías en el linker:
   - `wintrust.lib`
   - `psapi.lib`
   - `ntdll.lib`
   - `ws2_32.lib`
   - `iphlpapi.lib`
2. Configura el proyecto para **x64**.
3. Ejecuta como **Administrador** (necesario para `SeDebugPrivilege`).

## 🔍 Detalles de la Lógica de Riesgo
El sistema asigna puntos basados en:
- **[HIDDEN]:** +45 puntos.
- **[SHELLCODE_THREAD]:** +80 puntos.
- **[SUSPICIOUS_RWX]:** +40 puntos.
- **[NET_ACTIVA]:** +40 puntos (si el proceso no está firmado).

---
*Desarrollado para investigación de seguridad y análisis forense.*
