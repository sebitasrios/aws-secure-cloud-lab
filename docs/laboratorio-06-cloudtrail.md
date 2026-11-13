# Laboratorio 06 — Auditoría con AWS CloudTrail

## Objetivo

Configurar y verificar el registro de actividades mediante AWS CloudTrail en AWS Emulator local.

## Entorno

- AWS Emulator mediante Docker.
- AWS CLI y PowerShell.
- Endpoint local: https://127.0.0.1:8085
- Región simulada: us-east-1.
- No se utilizaron recursos de AWS real.

## 1. Consulta inicial

Se ejecutaron describe-trails, list-trails y lookup-events.

Resultados:
- Inicialmente no existían trails configurados.
- Se encontraron eventos históricos de operaciones IAM.
- Entre las operaciones registradas estaban CreateAccessKey, DeleteAccessKey y PutUserPolicy.
- Los eventos incluían usuario, fecha, servicio y tipo de operación.

## 2. Configuración del bucket S3

Se creó el bucket secure-cloud-audit-logs.

Se habilitó el bloqueo de acceso público y se configuró una política para permitir que CloudTrail consultara la ACL y escribiera registros en el prefijo AWSLogs/000000000000/.

El primer intento de creación del trail produjo InsufficientS3BucketPolicyException.

Después de corregir la política, la creación fue exitosa.

## 3. Creación y activación del trail

Nombre: secure-cloud-trail

ARN:
arn:aws:cloudtrail:us-east-1:000000000000:trail/secure-cloud-trail

Bucket de destino: secure-cloud-audit-logs

Configuración observada:
- IncludeGlobalServiceEvents: true
- IsMultiRegionTrail: false
- LogFileValidationEnabled: false

Se ejecutó start-logging correctamente.

## 4. Verificación

La operación get-trail-status devolvió:

- IsLogging: true
- LatestDeliveryTime: 2026-11-13T18:39:14.157200-05:00
- LatestDeliveryError: null

## 5. Evidencia de almacenamiento

Se consultó el bucket mediante list-objects-v2.

Se encontraron dos archivos .json.gz en:

AWSLogs/000000000000/CloudTrail/us-east-1/2026/11/13/

Esto demuestra que el emulador generó objetos de auditoría en S3.

No se verificó el contenido interno ni la integridad criptográfica de los archivos.

## 6. Importancia para la seguridad

CloudTrail permite consultar acciones administrativas, investigar modificaciones y analizar operaciones relacionadas con identidades y credenciales.

La auditoría ayuda a reconstruir acontecimientos durante un incidente de seguridad.

## 7. Limitaciones

Las pruebas se realizaron exclusivamente en AWS Emulator.

No se verificó una infraestructura real de AWS, la integridad criptográfica ni la retención a largo plazo de los registros.

## Conclusión

Se configuró un trail, se corrigieron permisos de S3, se activó el registro y se comprobó la existencia de archivos de auditoría en el bucket del emulador.
