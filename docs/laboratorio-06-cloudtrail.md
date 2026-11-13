
## 7. Verificación de archivos en S3

Se consultaron los objetos almacenados en el bucket secure-cloud-audit-logs.

Resultado: se encontraron dos archivos de auditoría con extensión .json.gz dentro de la ruta AWSLogs/000000000000/CloudTrail/us-east-1/2026/11/13/.

Esto confirma que AWS Emulator generó objetos de registro asociados al trail.

No se verificó el contenido interno ni la integridad criptográfica de dichos archivos.
