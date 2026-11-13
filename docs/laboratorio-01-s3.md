# Laboratorio 01 — Seguridad en Amazon S3

## Objetivo
Configurar y verificar una conexión segura entre AWS CLI y AWS Emulator para realizar pruebas de seguridad en S3.

## Entorno
- Sistema operativo: Windows 11
- Contenedores: Docker
- Emulador: AWS Emulator
- Herramienta: AWS CLI
- Región: us-east-1
- Endpoint local: https://127.0.0.1:8085
- Perfil: aws-emulator

## Pruebas realizadas

### 1. Estado del contenedor
Se verificó que el contenedor `aws-secure-lab` estaba en ejecución mediante `docker ps`.

### 2. Conexión HTTPS
Se consultó el endpoint `/__admin/health`, obteniendo `HTTP 200 OK`.

### 3. Autenticación
Se configuró AWS CLI con las credenciales locales generadas por AWS Emulator.

### 4. Validación del certificado
Se configuró el certificado de la autoridad certificadora local mediante `--ca-bundle`, evitando el uso de `--no-verify-ssl`.

### 5. Consulta de S3
Comando ejecutado:

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" s3api list-buckets
```

Resultado: consulta exitosa, sin buckets existentes.

## Conclusión
Se verificó la conectividad HTTPS, la autenticación del emulador y la respuesta de la API S3. Todavía no se han probado políticas IAM ni controles de acceso a objetos.

## Próxima actividad
Crear un bucket S3 y comprobar los controles de privacidad disponibles en AWS Emulator.

## Resultados de las pruebas S3

### Creación del bucket
Se creó correctamente el bucket `seminario-seguridad-s3` mediante AWS CLI.

### Configuración de acceso público
La consulta `get-public-access-block` devolvió los cuatro controles configurados en `true`:

- BlockPublicAcls
- IgnorePublicAcls
- BlockPublicPolicy
- RestrictPublicBuckets

### Almacenamiento y cifrado
Se subió el archivo `prueba-s3.txt` y se verificó su existencia mediante `list-objects-v2`.

La respuesta del emulador informó cifrado del lado del servidor `AES256` (SSE-S3).

### Prueba de acceso anónimo
Se intentó descargar el archivo mediante una solicitud sin firma (`--no-sign-request`).

**Resultado:** `AccessDenied`.

### Prueba de acceso autenticado
Se descargó el mismo archivo utilizando el perfil `aws-emulator`.

**Resultado:** operación exitosa. Se comprobó que el contenido descargado coincide con el archivo original.

### Conclusión de seguridad
AWS Emulator rechazó la solicitud anónima y permitió la solicitud autenticada. Estos resultados corresponden al comportamiento observado en el entorno simulado y no constituyen una validación completa de los controles de AWS real.

### Próxima etapa
Estudiar IAM y el principio de mínimo privilegio, verificando qué operaciones y restricciones permite simular AWS Emulator.
