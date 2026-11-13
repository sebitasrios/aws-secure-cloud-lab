# Laboratorio 02 — IAM y principio de mínimo privilegio

## Objetivo

Implementar y comprobar una política IAM que limite las operaciones de un usuario sobre un bucket S3 en AWS Emulator.

## Configuración realizada

- Usuario IAM: `analista-s3`
- Política en línea: `PoliticaLecturaS3`
- Archivo de configuración: `policies/s3-read-only.json`
- Bucket: `seminario-seguridad-s3`
- Región: `us-east-1`

## Permisos configurados

La política concede únicamente las siguientes acciones:

- `s3:ListBucket`: listar objetos del bucket.
- `s3:GetObject`: descargar objetos del bucket.

No concede permisos para subir ni eliminar objetos.

## Pruebas realizadas

| Prueba | Resultado |
|---|---|
| Crear usuario IAM | Exitosa |
| Registrar y consultar la política | Exitosa |
| Listar objetos del bucket | Permitido |
| Descargar `prueba-s3.txt` | Permitido |
| Intentar eliminar `prueba-eliminacion-iam.txt` | AccessDenied |

## Evidencia de seguridad

Al intentar eliminar un objeto con el usuario `analista-s3`, AWS Emulator respondió:

`AccessDenied`

El mensaje indicó que ninguna política de identidad autorizaba la acción `s3:DeleteObject`.

## Conclusión

Las pruebas demostraron que, para las operaciones evaluadas, AWS Emulator permitió las acciones de lectura configuradas y rechazó una operación de eliminación no autorizada.

Esto ilustra el principio de mínimo privilegio: una identidad debe contar solamente con los permisos necesarios para cumplir su función.

Los resultados corresponden a un entorno simulado y no constituyen una validación completa del comportamiento de seguridad de AWS real.