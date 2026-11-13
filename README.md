
# AWS Secure Cloud Lab

**Proyecto práctico de seguridad en la nube | AWS Emulator | Docker | AWS CLI | Ciberseguridad**

AWS Secure Cloud Lab es un proyecto educativo que implementa y evalúa controles básicos de seguridad en una infraestructura AWS simulada localmente.

El proyecto aborda la protección de datos, gestión de identidades, segmentación de redes, seguridad de instancias, cifrado, auditoría y detección de amenazas.

> **Importante:** todos los laboratorios fueron desarrollados utilizando AWS Emulator ejecutado localmente mediante Docker. No se utilizaron servicios de AWS real ni se generaron costos de infraestructura en la nube.

---

## 1. Pregunta central del proyecto

**¿Cómo impedir que una mala configuración o una credencial comprometida permita tomar control de una carga de trabajo en AWS?**

La propuesta consiste en aplicar múltiples controles de seguridad para reducir los riesgos asociados con configuraciones inseguras, permisos excesivos y accesos no autorizados.

Se utiliza el principio de **defensa en profundidad**, que consiste en implementar diferentes capas de protección en lugar de depender de un único mecanismo de seguridad.

## 2. Objetivo general

Implementar y comprobar controles básicos de seguridad en una infraestructura AWS simulada, aplicando principios de mínimo privilegio, segmentación de redes, protección de datos, auditoría y detección de amenazas.

### Objetivos específicos

- Proteger el almacenamiento de archivos contra accesos públicos no autorizados.
- Implementar permisos mínimos mediante IAM.
- Configurar redes y grupos de seguridad para separar componentes.
- Aplicar controles de seguridad en instancias EC2 y volúmenes EBS.
- Utilizar KMS para cifrar y descifrar información.
- Registrar actividades administrativas mediante CloudTrail.
- Generar y consultar hallazgos simulados con GuardDuty.

## 3. Tecnologías utilizadas

| Tecnología | Función |
|---|---|
| AWS Emulator | Simulación local de servicios AWS |
| Docker | Ejecución del entorno emulado |
| AWS CLI | Administración de recursos mediante comandos |
| PowerShell | Automatización y ejecución de pruebas |
| Git | Control de versiones |
| GitHub | Publicación y documentación del proyecto |
| Amazon S3 | Almacenamiento y protección de archivos |
| AWS IAM | Gestión de identidades y permisos |
| Amazon VPC | Configuración y segmentación de redes |
| Amazon EC2 | Simulación de instancias de cómputo |
| Amazon EBS | Almacenamiento asociado a instancias |
| AWS KMS | Administración de claves y cifrado |
| AWS CloudTrail | Auditoría de actividades |
| Amazon GuardDuty | Detección y consulta de hallazgos |

## 4. Arquitectura del laboratorio

Se configuró una VPC simulada con la siguiente estructura:

| Componente | Configuración |
|---|---|
| VPC | 10.10.0.0/16 |
| Subred de aplicación | 10.10.1.0/24 |
| Subred de base de datos | 10.10.2.0/24 |
| Aplicación | Puerto HTTPS 443 |
| Base de datos | Puerto PostgreSQL 5432 |
| Instancia EC2 | t3.micro |
| IP privada de EC2 | 10.10.1.4 |

### Segmentación de red

La arquitectura contempla dos segmentos principales:

**Subred de aplicación**

Destinada a alojar componentes que prestan servicios a los usuarios.

**Subred de base de datos**

Destinada a separar los recursos de almacenamiento de datos de la capa de aplicación.

Se configuraron grupos de seguridad para permitir HTTPS hacia la aplicación y restringir el acceso PostgreSQL al grupo de seguridad de la aplicación.

Estas reglas fueron configuradas en el emulador, pero no se realizaron pruebas completas de aislamiento de tráfico real.

## 5. Laboratorios desarrollados

El proyecto está organizado en siete laboratorios.

### Laboratorio 01 — Seguridad en Amazon S3

**Objetivo:** proteger un bucket contra accesos públicos no autorizados.

Actividades realizadas:

- Creación de un bucket S3.
- Configuración del bloqueo de acceso público.
- Verificación de permisos.
- Prueba de acceso anónimo.

**Resultado:** el acceso anónimo fue denegado y el almacenamiento quedó configurado con restricciones de acceso público.

[Ver documentación del Laboratorio 01](docs/laboratorio-01-s3.md)

### Laboratorio 02 — Seguridad con AWS IAM

**Objetivo:** aplicar el principio de mínimo privilegio.

Actividades realizadas:

- Creación del usuario `analista-s3`.
- Configuración de permisos limitados sobre S3.
- Prueba de operaciones autorizadas.
- Intento de eliminación de recursos sin permisos suficientes.

**Resultado:** se verificó que el usuario podía realizar operaciones permitidas y que la eliminación de recursos era denegada.

[Ver documentación del Laboratorio 02](docs/laboratorio-02-iam.md)

### Laboratorio 03 — Seguridad de red con Amazon VPC

**Objetivo:** configurar una arquitectura de red segmentada.

Actividades realizadas:

- Creación de una VPC.
- Configuración de subredes de aplicación y base de datos.
- Configuración de Internet Gateway y rutas.
- Creación de grupos de seguridad.
- Configuración de reglas para HTTPS y PostgreSQL.

**Resultado:** se estableció una estructura de red con reglas de acceso diferenciadas.

[Ver documentación del Laboratorio 03](docs/laboratorio-03-vpc.md)

### Laboratorio 04 — Seguridad de EC2 y EBS

**Objetivo:** aplicar controles de seguridad a instancias y almacenamiento.

Actividades realizadas:

- Creación de una instancia EC2 simulada.
- Configuración de IMDSv2 obligatorio.
- Inspección de un volumen EBS.
- Activación del cifrado predeterminado de EBS.
- Creación de un volumen de prueba cifrado.
- Creación y copia cifrada de snapshots.

**Resultado:** se comprobó el uso de IMDSv2 y el cifrado de un nuevo volumen de prueba.

**Limitación:** el volumen raíz original permaneció sin cifrar. El emulador presentó un error al intentar crear un volumen desde una copia cifrada del snapshot.

[Ver documentación del Laboratorio 04](docs/laboratorio-04-ec2.md)

### Laboratorio 05 — Cifrado con AWS KMS

**Objetivo:** proteger información mediante operaciones criptográficas.

Actividades realizadas:

- Creación de una clave KMS.
- Verificación de su estado.
- Cifrado de un mensaje de prueba.
- Descifrado del mensaje.
- Comprobación de recuperación del contenido original.

**Resultado:** las operaciones de cifrado y descifrado se completaron correctamente.

No se realizaron pruebas de restricciones de acceso a la clave.

[Ver documentación del Laboratorio 05](docs/laboratorio-05-kms.md)

### Laboratorio 06 — Auditoría con AWS CloudTrail

**Objetivo:** registrar actividades administrativas para facilitar investigaciones de seguridad.

Actividades realizadas:

- Consulta de eventos históricos.
- Creación de un bucket S3 para auditoría.
- Configuración de permisos para CloudTrail.
- Creación del trail `secure-cloud-trail`.
- Activación del registro.
- Verificación de su estado.
- Consulta de archivos de auditoría almacenados en S3.

**Resultado:** el trail quedó activo y se verificó la generación de dos archivos de registro `.json.gz` en el bucket de auditoría.

No se validó la integridad criptográfica de los archivos.

[Ver documentación del Laboratorio 06](docs/laboratorio-06-cloudtrail.md)

### Laboratorio 07 — Detección de amenazas con Amazon GuardDuty

**Objetivo:** configurar un detector y consultar hallazgos de seguridad simulados.

Actividades realizadas:

- Consulta inicial de detectores.
- Creación de un detector GuardDuty.
- Verificación del estado `ENABLED`.
- Consulta inicial de hallazgos.
- Generación de hallazgos de ejemplo.
- Consulta de identificadores de hallazgos generados.

**Resultado:** se habilitó el detector y se generaron y consultaron múltiples hallazgos simulados.

Los hallazgos fueron creados artificialmente mediante `create-sample-findings`; no representan ataques reales detectados.

[Ver documentación del Laboratorio 07](docs/laboratorio-07-guardduty.md)

## 6. Configuración del entorno local

### Requisitos

Para reproducir las pruebas se necesita:

- Docker instalado y funcionando.
- AWS Emulator configurado y en ejecución.
- AWS CLI instalado.
- PowerShell.
- Perfil de AWS CLI configurado para el emulador.
- Certificado CA local del emulador.

### Configuración utilizada

El entorno local empleó el siguiente endpoint:

`https://127.0.0.1:8085`

Ejemplo de conexión desde PowerShell:

```powershell
$AWS = "aws"
$ENDPOINT = "https://127.0.0.1:8085"
$CA = "$env:USERPROFILE\.aws\emulator-ca.pem"

& $AWS --profile aws-emulator --endpoint-url $ENDPOINT --ca-bundle $CA s3api list-buckets --no-cli-pager
```

Este comando consulta los buckets S3 disponibles en el emulador.

**Advertencia:** los ejemplos están diseñados para el entorno local. No deben ejecutarse directamente contra una cuenta AWS real sin una revisión previa.

## 7. Resultados generales

| Control | Resultado obtenido |
|---|---|
| Protección de S3 | Acceso anónimo denegado |
| Mínimo privilegio IAM | Operación de eliminación denegada |
| Segmentación VPC | Subredes y grupos de seguridad configurados |
| Protección EC2 | IMDSv2 obligatorio |
| Cifrado EBS | Cifrado predeterminado habilitado y volumen de prueba cifrado |
| Cifrado KMS | Cifrado y descifrado exitosos |
| Auditoría CloudTrail | Trail activo y archivos almacenados en S3 |
| GuardDuty | Detector habilitado y hallazgos simulados generados |

## 8. Limitaciones del proyecto

Este proyecto se desarrolló exclusivamente en un entorno educativo de simulación.

Por lo tanto:

- No se desplegó infraestructura en AWS real.
- No se realizaron pruebas de penetración.
- No se comprobó el aislamiento efectivo de todos los segmentos de red.
- El volumen raíz original de EC2 permaneció sin cifrar.
- No se verificó la integridad criptográfica de los archivos CloudTrail.
- Los hallazgos GuardDuty fueron simulados.
- No se demostró detección automática de ataques reales.
- Los resultados del emulador no garantizan un comportamiento idéntico en AWS real.

Estas limitaciones se documentan para distinguir las configuraciones realizadas de los controles efectivamente comprobados.

## 9. Aprendizajes obtenidos

El desarrollo de los laboratorios permitió practicar conceptos fundamentales de seguridad en la nube:

**Mínimo privilegio:** los usuarios deben contar únicamente con los permisos necesarios para realizar sus funciones.

**Defensa en profundidad:** diferentes controles de seguridad pueden complementarse para reducir riesgos.

**Segmentación de redes:** separar componentes permite definir reglas de comunicación entre ellos.

**Protección de datos:** el cifrado contribuye a proteger la confidencialidad de la información.

**Auditoría:** registrar actividades permite investigar acciones realizadas sobre los recursos.

**Detección de amenazas:** los hallazgos de seguridad pueden apoyar los procesos de análisis y respuesta a incidentes.

## 10. Conclusión

AWS Secure Cloud Lab demuestra, mediante siete laboratorios simulados, cómo diferentes controles pueden combinarse para reducir riesgos de seguridad en una infraestructura de nube.

El proyecto permitió practicar configuraciones de almacenamiento, identidades, redes, cómputo, cifrado, auditoría y detección de amenazas.

También permitió identificar limitaciones técnicas y comprender que configurar un control no equivale necesariamente a comprobar su efectividad ante ataques reales.

El resultado es una base práctica para continuar aprendiendo sobre seguridad en la nube, desarrollo backend y ciberseguridad.

---

## Autor

**Sebastián Ríos**

Estudiante de Tecnología en Desarrollo de Software.

Interesado en desarrollo backend, seguridad informática, redes y tecnologías cloud.

**GitHub:** https://github.com/sebitasrios

**LinkedIn:** https://www.linkedin.com/in/sebastianriosdev/

---

*Proyecto desarrollado con fines educativos utilizando AWS Emulator local y Docker.*
