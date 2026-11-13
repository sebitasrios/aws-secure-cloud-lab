# Laboratorio 03 — Seguridad de red con VPC y Security Groups

## 1. Objetivo

Diseñar y configurar una arquitectura de red en AWS Emulator que permita separar una aplicación web de una base de datos, aplicando principios de segmentación de red, mínimo privilegio y reducción de la superficie de exposición.

Este laboratorio forma parte del proyecto **AWS Secure Cloud Lab**, orientado a responder la pregunta:

**¿Cómo impedir que una mala configuración o una credencial comprometida permita tomar control de una carga de trabajo en AWS?**

## 2. Herramientas utilizadas

- AWS Emulator ejecutado mediante Docker.
- AWS CLI para administrar los recursos.
- PowerShell en Windows.
- Visual Studio Code.
- Git y GitHub para documentar el proyecto.

## 3. Arquitectura de red

Se creó una VPC con el bloque de direcciones `10.10.0.0/16`, dividida en dos subredes:

| Recurso | Identificador | Configuración |
|---|---|---|
| VPC | `vpc-00000000000000080` | `10.10.0.0/16` |
| Subred de aplicación | `subnet-00000000000000088` | `10.10.1.0/24` |
| Subred de base de datos | `subnet-0000000000000008a` | `10.10.2.0/24` |
| Security Group de aplicación | `sg-0000000000000008c` | `aplicacion-web` |
| Security Group de base de datos | `sg-0000000000000008f` | `base-datos` |
| Tabla de rutas principal | `rtb-00000000000000083` | Ruta local |
| Tabla de rutas de aplicación | `rtb-00000000000000092` | Ruta local y ruta a Internet |
| Internet Gateway | `igw-00000000000000094` | Asociado a la VPC |

### Diseño de seguridad

**Subred de aplicación**

- Red: `10.10.1.0/24`.
- Tiene una ruta `0.0.0.0/0` hacia el Internet Gateway.
- Tiene habilitada la asignación automática de IP pública para futuras instancias.
- Su Security Group permite conexiones entrantes TCP por el puerto `443` desde `0.0.0.0/0`.

**Subred de base de datos**

- Red: `10.10.2.0/24`.
- Utiliza la tabla de rutas principal, sin ruta directa hacia Internet.
- Tiene deshabilitada la asignación automática de IP pública.
- Su Security Group permite conexiones entrantes TCP por el puerto `5432` únicamente desde el Security Group de aplicación.

Los Security Groups conservan las reglas de salida predeterminadas que permiten tráfico IPv4 saliente.

## 4. Configuración de los Security Groups

Se definieron dos grupos de seguridad para controlar el tráfico entrante.

| Grupo | Puerto | Origen permitido | Propósito |
|---|---|---|---|
| `aplicacion-web` | TCP 443 | `0.0.0.0/0` | Permitir HTTPS |
| `base-datos` | TCP 5432 | `sg-0000000000000008c` | Permitir PostgreSQL desde la aplicación |

La regla de PostgreSQL utiliza como origen otro Security Group, en lugar de permitir conexiones desde cualquier dirección IP.

Esta configuración está diseñada para reducir la exposición de la base de datos.

## 5. Configuración de las tablas de rutas

Se utilizaron dos tablas de rutas.

**Tabla de aplicación: `rtb-00000000000000092`**

| Destino | Gateway |
|---|---|
| `10.10.0.0/16` | `local` |
| `0.0.0.0/0` | `igw-00000000000000094` |

Asociada a la subred `subnet-00000000000000088`.

**Tabla principal: `rtb-00000000000000083`**

| Destino | Gateway |
|---|---|
| `10.10.0.0/16` | `local` |

La subred de base de datos utiliza esta tabla principal y no dispone de una ruta directa hacia Internet.

## 6. Comandos de verificación

Los siguientes comandos se ejecutan desde PowerShell con el perfil `aws-emulator`.

### Verificar la VPC

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 describe-vpcs --vpc-ids vpc-00000000000000080
```

### Verificar las subredes

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 describe-subnets --subnet-ids subnet-00000000000000088 subnet-0000000000000008a --query "Subnets[*].{SubnetId:SubnetId,CIDR:CidrBlock,PublicIP:MapPublicIpOnLaunch}" --output table
```

### Verificar los Security Groups

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 describe-security-groups --group-ids sg-0000000000000008c sg-0000000000000008f
```

### Verificar las tablas de rutas

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-00000000000000080"
```

### Verificar el Internet Gateway

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 describe-internet-gateways --internet-gateway-ids igw-00000000000000094
```

## 7. Resultados obtenidos

Durante el laboratorio se verificó mediante AWS CLI que:

1. La VPC y las dos subredes fueron creadas.
2. Los Security Groups contienen las reglas de entrada configuradas.
3. La tabla de aplicación está asociada a la subred `10.10.1.0/24`.
4. El Internet Gateway está asociado a la VPC y figura en estado `available`.
5. La tabla de aplicación tiene una ruta activa `0.0.0.0/0` hacia el Internet Gateway.
6. La tabla principal de la base de datos conserva únicamente la ruta local.
7. La subred de aplicación tiene `MapPublicIpOnLaunch = True`.
8. La subred de base de datos tiene `MapPublicIpOnLaunch = False`.

**Limitación de las pruebas:** estas verificaciones confirman la configuración registrada en AWS Emulator. No se desplegaron instancias EC2 ni se realizaron pruebas de tráfico entre servidores. Por tanto, no se ha demostrado todavía el aislamiento efectivo de la red.

## 8. Relación con la ciberseguridad

Este laboratorio aplica tres principios:

**Segmentación de red:** separar la aplicación y la base de datos en subredes diferentes.

**Mínima exposición:** evitar una ruta directa hacia Internet desde la subred de base de datos.

**Mínimo privilegio de red:** permitir el acceso entrante al puerto PostgreSQL únicamente desde el Security Group de la aplicación.

Estas medidas forman parte de una estrategia de defensa en profundidad. Por sí solas no eliminan el riesgo de credenciales comprometidas ni sustituyen otros controles de seguridad.

## 9. Conclusión

Se configuró y verificó mediante AWS CLI una arquitectura de red con una VPC, dos subredes, dos Security Groups, dos tablas de rutas y un Internet Gateway.

La subred de aplicación quedó preparada para alojar recursos con conectividad pública, mientras que la subred de base de datos quedó configurada sin ruta directa hacia Internet.

El laboratorio demuestra cómo utilizar controles de red de AWS para reducir la exposición de recursos. Como siguiente etapa, será necesario desplegar cargas de trabajo y realizar pruebas de conectividad para evaluar el comportamiento efectivo de estos controles.