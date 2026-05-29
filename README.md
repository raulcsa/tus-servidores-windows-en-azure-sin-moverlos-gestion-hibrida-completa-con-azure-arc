# Gestión Híbrida de Windows Servers en Azure sin moverlos

Este repositorio contiene una guía práctica y detallada para implementar una solución de **gestión híbrida completa** de servidores locales (físicos o virtuales) Windows Server utilizando **Azure Arc** y la suite de servicios de gobernanza, seguridad y operaciones de Microsoft Azure.

---

## Descripción del Proyecto

El objetivo de este proyecto es demostrar cómo extender el plano de control de Azure hacia el centro de datos local (on-premises) para administrar servidores Windows existentes de forma unificada, sin necesidad de migrarlos a la nube.

La guía principal documenta el proceso paso a paso para configurar:
*   La conexión de servidores on-premises mediante el **agente de computación híbrida de Azure Arc**.
*   La sincronización del Directorio Activo (AD DS) local con la nube usando **Microsoft Entra Connect**.
*   La protección avanzada contra amenazas con **Microsoft Defender for Servers** (Defender for Cloud).
*   El control de cumplimiento normativo y gobernanza con **Azure Policy** (Guest Configuration).
*   La monitorización unificada y alertas con **Azure Monitor** (Log Analytics y DCR).
*   La detección y respuesta ante incidentes (SIEM/SOAR) mediante **Microsoft Sentinel**.
*   El control centralizado de parches de seguridad con **Azure Update Manager**.
*   La protección de datos críticos y estado del sistema (System State) con **Azure Backup**.

---

##  Arquitectura de la Solución

El entorno híbrido implementado consta de los siguientes componentes:

*   **SRV-01 (Local):** Servidor Windows Server que actúa como Domain Controller principal para Active Directory local.
*   **SRV-02 (Local):** Servidor Windows Server miembro del dominio que hospeda servicios de base de datos (SQL Server / MySQL) y aplicaciones de producción.
*   **Microsoft Entra ID:** Proveedor de identidad en la nube sincronizado localmente con `SRV-01` mediante Entra Connect.
*   **Azure Arc Control Plane:** Canal de administración HTTPS saliente (puerto 443) que expone los servidores locales como recursos híbridos administrados en Azure.

---

## Contenido del Repositorio

*   **[Gestion_Hibrida_Windows_Servers.md](file:///home/raul/tus-servidores-windows-en-azure-sin-moverlos-gestion-hibrida-completa-con-azure-arc/Gestion_Hibrida_Windows_Servers.md):** La guía de implementación completa paso a paso, dividida en 11 módulos con explicaciones técnicas detalladas de cada servicio y comandos de consola.
*   **[media/](file:///home/raul/tus-servidores-windows-en-azure-sin-moverlos-gestion-hibrida-completa-con-azure-arc/media/):** Recursos visuales, capturas del portal de Azure y diagramas de arquitectura de red.

---

## Requisitos Previos

Para seguir la guía de implementación, necesitarás:
1.  Una suscripción activa de Microsoft Azure con privilegios para registrar Resource Providers y asignar roles RBAC.
2.  Acceso de administrador en un tenant de Microsoft Entra ID.
3.  Dos servidores locales Windows Server (físicos o máquinas virtuales en Hyper-V/VMware) con conectividad a Internet.
4.  Un bosque local de Active Directory Domain Services (AD DS) configurado.
