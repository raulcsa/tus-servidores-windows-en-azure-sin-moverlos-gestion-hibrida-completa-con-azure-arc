# Gestión Híbrida de Windows Servers

**Guía Completa de Implementación Paso a Paso**

# Índice

- [MÓDULO 0: Arquitectura del entorno](#módulo-0-arquitectura-del-entorno)
- [MÓDULO 1: PREPARACIÓN DE AZURE](#módulo-1-preparación-de-azure)
  - [1.1 Crear el Resource Group en Norway East](#11-crear-el-resource-group-en-norway-east)
  - [1.2 Registrar Proveedores de Recursos de Azure Arc](#12-registrar-proveedores-de-recursos-de-azure-arc)
- [MÓDULO 2: MICROSOFT ENTRA ID](#módulo-2-microsoft-entra-id)
  - [2.1 Conceptos Clave de Entra ID](#21-conceptos-clave-de-entra-id-en-este-proyecto)
  - [2.2 Verificar el Tenant de Entra ID](#22-verificar-el-tenant-de-entra-id)
  - [2.3 Crear Service Principal para Azure Arc](#23-crear-service-principal-para-azure-arc)
  - [2.4 Preparar Active Directory en SRV-01 para Entra ID](#24-preparar-active-directory-en-srv-01-para-entra-id)
- [MÓDULO 3: INSTALACIÓN DEL AGENTE AZURE ARC](#módulo-3-instalación-del-agente-azure-arc)
- [MÓDULO 4: ENTRA CONNECT (Azure AD Connect)](#módulo-4-entra-connect-azure-ad-connect)

# MÓDULO 0: Arquitectura del entorno

![Diagrama El contenido generado por IA puede ser
incorrecto.](media/image1.png)

# MÓDULO 1: PREPARACIÓN DE AZURE

Crear Resource Group, permisos y configuración inicial

## 1.1 Crear el Resource Group en Norway East

Todos los recursos del proyecto deben desplegarse en Norway East.
Comenzamos creando el Resource Group que contendrá todo.

### Configuración del Resource Group

![](media/image2.png)

## 1.2 Registrar Proveedores de Recursos de Azure Arc

Antes de poder conectar servidores a Azure Arc, debemos registrar los
resource providers en mi suscripción.

Desde Azure Cloud Shell:

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.HybridCompute

Register-AzResourceProvider -ProviderNamespace
Microsoft.GuestConfiguration

Register-AzResourceProvider -ProviderNamespace
Microsoft.HybridConnectivity

Register-AzResourceProvider -ProviderNamespace Microsoft.AzureArcData

Register-AzResourceProvider -ProviderNamespace
Microsoft.OperationsManagement

Register-AzResourceProvider -ProviderNamespace Microsoft.Insights
```



![](media/image3.png)

# MÓDULO 2: MICROSOFT ENTRA ID

Integración de identidades híbridas con Active Directory

## 2.1 Conceptos Clave de Entra ID en este Proyecto

Microsoft Entra ID (anteriormente Azure AD) es el servicio de identidad
en la nube. En este proyecto lo usaremos para:

-   Sincronizar las cuentas de Active Directory de SRV-01 a la nube
    mediante Entra Connect

-   Autenticar usuarios de SRV-01 en servicios de Azure

-   Gestionar el acceso a los recursos de Azure Arc con RBAC

-   Aplicar Conditional Access sobre los administradores del entorno

## 2.2 Verificar el Tenant de Entra ID

\# En Azure Cloud Shell - Verificar el tenant asociado a mi suscripción

Get-AzContext

\# Obtener información del tenant

Get-AzTenant

![](media/image4.png)

## 2.3 Crear Service Principal para Azure Arc

Azure Arc necesita una identidad (Service Principal) para autenticar los
servidores contra Azure. Esta es la práctica recomendada para entornos
de producción.

### 2.3.1: Crear el Service Principal

\# Crear Service Principal específico para Azure Arc

\$subId = (Get-AzContext).Subscription.Id

\$sp = New-AzADServicePrincipal -DisplayName \"sp-azure-arc-onboarding\"
\`

-Role \"Azure Connected Machine Onboarding\" \`

-Scope \"/subscriptions/\$subId\"

\$sp.AppId

\# Crear credencial para el SP

\$spCred = New-AzADSpCredential -ObjectId \$sp.Id

\$spCred.SecretText

![](media/image5.png)

![](media/image6.png)

### 2.3.2: Asignar rol adicional

\# Asignar rol de lector al Resource Group para el SP

\$subId = (Get-AzContext).Subscription.Id

New-AzRoleAssignment -ApplicationId \$sp.AppId \`

-RoleDefinitionName \'Reader\' \`

-Scope \"/subscriptions/\$subId/resourceGroups/rg-winservers-arc\"

![](media/image7.png)

## 2.4 Preparar Active Directory en SRV-01 para Entra ID

### 2.4.1: Verificar el estado del Domain Controller

\# Verificar que AD DS está funcionando correctamente

Get-Service NTDS, ADWS, DNS \| Select-Object Name, Status

\# Verificar el nombre del dominio

Get-ADDomain \| Select-Object DNSRoot, NetBIOSName, DomainMode

\# Listar usuarios del dominio (para planificar la sincronización)

Get-ADUser -Filter \* \| Select-Object Name, SamAccountName, Enabled \|
Format-Table

\# Verificar UPN suffixes configurados

Get-ADForest \| Select-Object UPNSuffixes

![](media/image8.png)

### 2.4.2: Crear UPN Suffix personalizado

Set-ADForest -Identity (Get-ADForest) -UPNSuffixes
\@{Add=\'raul6544.onmicrosoft.com\'}

Get-ADForest \| Select-Object -ExpandProperty UPNSuffixes

![](media/image9.png)

# MÓDULO 3: INSTALACIÓN DEL AGENTE AZURE ARC

Conectar SRV-01 y SRV-02 a Azure Arc

## 3.1 Generar Script de Instalación desde el Portal

La forma más sencilla de instalar el agente es generar el script de
onboarding directamente desde el portal de Azure.

![](media/image10.png)

![](media/image11.png)

![](media/image12.png)

![](media/image13.png)

![](media/image14.png)

![](media/image15.png)

![](media/image16.png)

![](media/image17.png)

# MÓDULO 4: ENTRA CONNECT (Azure AD Connect)

### 4.1.1: Descargar e instalar Entra Connect

![](media/image18.png)

![](media/image19.png)

### 4.1.3: Verificar la sincronización

![](media/image20.png)

![](media/image21.png)

# MÓDULO 5: MICROSOFT DEFENDER FOR SERVERS

Protección avanzada para SRV-01 y SRV-02

## 5.1 Habilitar Defender for Cloud

### 5.1.1: Acceder a Defender for Cloud

![](media/image22.png)

![](media/image23.png)

## 5.2 Instalar el Agente de Monitorización (MMA/AMA)

Defender for Servers requiere el Azure Monitor Agent (AMA) en los
servidores para recolectar datos de seguridad.

### 5.2.1: Instalar AMA en SRV-01y SRV-02 vía extensión de Arc

![](media/image24.png)

![](media/image25.png)

![](media/image26.png)

### 5.2.2: Recomendaciones de Defender for Cloudg

![](media/image27.png)

# MÓDULO 6: AZURE POLICY

Políticas personalizadas de gobernanza y compliance

## 6.1 Conceptos y Estrategia de Políticas

Azure Policy garantiza que los recursos cumplan con los estándares
corporativos. Para este proyecto crearemos políticas específicas para
los servidores Arc.

  -----------------------------------------------------------------------------------
  **Política**             **Tipo**            **Afecta a**   **Objetivo**
  ------------------------ ------------------- -------------- -----------------------
  Require-AzureArc-Agent   Deny/Audit          SRV-01, SRV-02 Verificar que el agente
                                                              está instalado y
                                                              conectado

  Audit-WindowsFirewall    Audit               SRV-01, SRV-02 Auditar que el firewall
                                                              de Windows está activo

  Deploy-AMA-Agent         DeployIfNotExists   SRV-01, SRV-02 Desplegar AMA
                                                              automáticamente si
                                                              falta

  Audit-AdminAccounts      Audit               SRV-01         Auditar cuentas de
                                                              admin locales en DC

  Require-MySQL-TLS        Audit               SRV-02         Verificar configuración
                                                              TLS en MySQL

  Audit-Updates-Pending    Audit               SRV-01, SRV-02 Detectar
                                                              actualizaciones
                                                              críticas pendientes
  -----------------------------------------------------------------------------------

![](media/image28.png)

![](media/image29.png)

![](media/image30.png)

# MÓDULO 7: AZURE MONITOR

Log Analytics, alertas y diagnósticos para ambos servidores

## 7.1 Crear Log Analytics Workspace

![](media/image31.png)

![](media/image32.png)

## 7.2 Crear Data Collection Rule (DCR)

Las Data Collection Rules definen qué datos recolectar de los servidores
Arc y dónde enviarlos.

### Paso 7.2.1: Crear DCR para eventos de Windows

![](media/image33.png)

![](media/image34.png)

![](media/image35.png)

![](media/image36.png)

![](media/image37.png)

![](media/image38.png)

![](media/image39.png)

![](media/image40.png)

## 7.3 Crear Alertas de Monitor

### Alerta 1: Agente Arc desconectado

![](media/image41.png)

![](media/image42.png)

![](media/image43.png)

![](media/image44.png)

![](media/image45.png)

![](media/image46.png)

![](media/image47.png)

### Alerta 2: CPU alta en cualquier servidor

![](media/image48.png)

![](media/image49.png)

### Consultas KQL útiles en Log Analytics

![](media/image50.png)

![](media/image51.png)

![](media/image52.png)

# MÓDULO 8: MICROSOFT SENTINEL

SIEM/SOAR para detección y respuesta a amenazas

## 8.1 Habilitar Microsoft Sentinel

![](media/image60.png)

## 8.2 Configurar Data Connectors

![](media/image67.png)![](media/image68.png)![](media/image75.png)

![](media/image53.png)![](media/image85.png)

##  

## 8.3 Habilitar Analytics Rules (Reglas de Detección)

### Reglas para SRV-01 (Active Directory)

![](media/image63.png)![](media/image86.png)![](media/image80.png)![](media/image82.png)

## 8.4 Crear Playbook (Automatización)

### Playbook: Bloqueo automático de IP sospechosa

![](media/image70.png)![](media/image57.png)![](media/image61.png)

# MÓDULO 9: AZURE UPDATE MANAGER

Gestión centralizada de actualizaciones y parches

## 9.1 Configurar Azure Update Manager

![](media/image66.png)![](media/image64.png)

# MÓDULO 10: AZURE BACKUP

Copias de seguridad para SRV-01 y SRV-02

## 10.1 Crear Recovery Services Vault

![](media/image73.png)![](media/image78.png)

## 10.2 Configurar Backup para SRV-01 (Active Directory)

### Paso 10.2.1: Instalar el agente MARS en SRV-01

![](media/image83.png)![](media/image71.png)![](media/image56.png)

![](media/image77.png)

### Paso 10.2.2: Configurar política de backup para AD

![](media/image72.png)![](media/image69.png)![](media/image84.png)![](media/image54.png)![](media/image74.png)![](media/image76.png)

## 10.3 Configurar Backup para SRV-02 (SQLServer)

![](media/image81.png)![](media/image58.png)![](media/image55.png)![](media/image79.png)![](media/image62.png)![](media/image59.png)![](media/image65.png)
