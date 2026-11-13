# Laboratorio 04 — Seguridad de instancias EC2 y cifrado EBS

## 1. Objetivo

Configurar una instancia EC2 en AWS Emulator aplicando controles de seguridad sobre la red, los metadatos y el almacenamiento.

El laboratorio busca identificar configuraciones inseguras, aplicar medidas preventivas y verificar los resultados mediante AWS CLI.

Forma parte del proyecto **AWS Secure Cloud Lab**, cuya pregunta principal es:

**¿Cómo impedir que una mala configuración o una credencial comprometida permita tomar control de una carga de trabajo en AWS?**

## 2. Herramientas utilizadas

- AWS Emulator ejecutado localmente mediante Docker.
- AWS CLI.
- PowerShell en Windows.
- Visual Studio Code.
- Git y GitHub.

**Alcance:** todas las operaciones se realizaron contra el emulador local, utilizando el endpoint `https://127.0.0.1:8085`. No se utilizaron recursos de AWS real.

## 3. Creación de la instancia EC2

Se consultaron las imágenes AMI disponibles en AWS Emulator y se seleccionó una imagen de Amazon Linux 2023.

También se verificó la disponibilidad del tipo de instancia `t3.micro`.

### Configuración utilizada

| Recurso | Configuración |
|---|---|
| Instancia | `i-00000000000000001` |
| AMI | `ami-0000000000000004c` |
| Sistema seleccionado | Amazon Linux 2023 |
| Tipo | `t3.micro` |
| CPU | 2 vCPU |
| Memoria | 1024 MiB |
| VPC | `vpc-00000000000000080` |
| Subred | `subnet-00000000000000088` |
| Security Group | `sg-0000000000000008c` |
| IP privada | `10.10.1.4` |
| IP pública reportada | `54.0.0.2` |
| Estado | `running` |

La instancia se creó utilizando la subred de aplicación configurada durante el Laboratorio 03.

La dirección IP pública fue reportada por el emulador y no demuestra conectividad real desde Internet.

## 4. Configuración de seguridad de EC2

### 4.1. Security Group

La instancia utiliza el Security Group `aplicacion-web`.

Su configuración de entrada permite:

| Protocolo | Puerto | Origen |
|---|---|---|
| TCP | 443 | `0.0.0.0/0` |

No se configuró una regla de entrada para SSH por el puerto 22.

Esta configuración busca reducir la superficie de exposición de la instancia.

### 4.2. Protección de metadatos con IMDSv2

Durante la creación de la instancia se configuró:

```text
HttpTokens=required
HttpEndpoint=enabled
```

Esto establece el uso obligatorio de IMDSv2 para las solicitudes al servicio de metadatos.

IMDSv2 utiliza tokens de sesión y ayuda a reducir determinados riesgos relacionados con el acceso no autorizado a metadatos de instancia.

La configuración fue verificada mediante AWS CLI, que devolvió:

```text
IMDSv2: required
```

## 5. Identificación de un volumen EBS sin cifrar

Se consultaron los volúmenes asociados a la instancia EC2.

### Resultado

| Propiedad | Valor |
|---|---|
| Volumen | `vol-00000000000000095` |
| Tamaño | 8 GiB |
| Estado | `in-use` |
| Dispositivo | `/dev/xvda` |
| Cifrado | `False` |

**Hallazgo de seguridad:** el volumen principal estaba registrado sin cifrado.

El cifrado en reposo es importante porque ayuda a proteger la información almacenada frente a determinados escenarios de acceso no autorizado.

## 6. Habilitación del cifrado EBS predeterminado

Se verificó inicialmente la configuración regional de cifrado EBS.

El resultado fue:

```json
{
  "EbsEncryptionByDefault": false
}
```

Se habilitó el cifrado predeterminado con el comando:

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 enable-ebs-encryption-by-default --no-cli-pager
```

Posteriormente se verificó nuevamente:

```json
{
  "EbsEncryptionByDefault": true
}
```

**Resultado:** AWS Emulator registró correctamente la activación del cifrado predeterminado.

Esta configuración está destinada a proteger nuevos volúmenes EBS. No cifra automáticamente los volúmenes existentes.

## 7. Prueba de creación de un volumen cifrado

Para verificar el comportamiento de la nueva configuración, se creó un volumen EBS de prueba de 1 GiB, sin especificar manualmente la opción de cifrado.

### Resultado

| Propiedad | Valor |
|---|---|
| Volumen | `vol-00000000000000099` |
| Tamaño | 1 GiB |
| Estado final | `available` |
| Cifrado | `True` |

**Conclusión de la prueba:** después de habilitar el cifrado predeterminado, AWS Emulator registró el nuevo volumen como cifrado.

Este resultado verifica los metadatos de configuración del emulador, pero no demuestra que exista cifrado físico de datos.

## 8. Prueba de migración mediante snapshots

Como el volumen original continuaba sin cifrar, se exploró un procedimiento de migración utilizando snapshots.

### 8.1. Snapshot del volumen original

Se creó un snapshot a partir de:

`vol-00000000000000095`

El emulador devolvió:

| Propiedad | Valor |
|---|---|
| Snapshot | `snap-0000000000000009a` |
| Estado final | `completed` |
| Cifrado | `False` |

### 8.2. Copia cifrada del snapshot

Posteriormente se solicitó una copia cifrada utilizando la opción `--encrypted`.

El resultado fue:

| Propiedad | Valor |
|---|---|
| Snapshot cifrado | `snap-0000000000000009c` |
| Estado final | `completed` |
| Cifrado | `True` |
| Volumen de origen reportado | `vol-0000000000000009b` |

La consulta confirmó que la copia estaba registrada como cifrada.

## 9. Limitación encontrada en AWS Emulator

Se intentó crear un nuevo volumen EBS utilizando el snapshot cifrado:

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 create-volume --snapshot-id snap-0000000000000009c --availability-zone us-east-1a --volume-type gp3 --encrypted
```

El emulador devolvió el siguiente error:

```text
InvalidSnapshot.NotFound
The snapshot 'snap-0000000000000009c' does not exist.
```

Sin embargo, al consultar nuevamente los snapshots, el emulador mostraba ambos registros:

| Snapshot | Estado | Cifrado |
|---|---|---|
| `snap-0000000000000009a` | `completed` | `False` |
| `snap-0000000000000009c` | `completed` | `True` |

### Interpretación

Existe una inconsistencia entre la consulta del snapshot y su utilización para crear un volumen.

Esto sugiere una posible limitación o un comportamiento particular de AWS Emulator. No se determinó la causa exacta.

Por esta razón, se decidió no continuar con la migración del volumen original.

**La instancia EC2 conservó su volumen principal sin modificaciones.**

## 10. Comandos de verificación

Todos los comandos utilizan exclusivamente el endpoint local de AWS Emulator.

### Consultar la instancia EC2

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 describe-instances --instance-ids i-00000000000000001 --query "Reservations[*].Instances[*].{ID:InstanceId,Estado:State.Name,Subred:SubnetId,SecurityGroups:SecurityGroups[*].GroupId,IMDSv2:MetadataOptions.HttpTokens}" --output json --no-cli-pager
```

### Verificar el cifrado EBS predeterminado

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 get-ebs-encryption-by-default --no-cli-pager
```

### Consultar los volúmenes

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 describe-volumes --query "Volumes[*].{ID:VolumeId,Estado:State,Cifrado:Encrypted,TamanoGiB:Size}" --output table --no-cli-pager
```

### Consultar los snapshots

```powershell
aws --profile aws-emulator --endpoint-url https://127.0.0.1:8085 --ca-bundle "$env:USERPROFILE\.aws\emulator-ca.pem" ec2 describe-snapshots --owner-ids self --query "Snapshots[*].{ID:SnapshotId,Estado:State,Cifrado:Encrypted}" --output table --no-cli-pager
```

## 11. Resultados del laboratorio

| Control o prueba | Resultado |
|---|---|
| Creación de instancia EC2 | Exitosa en el emulador |
| Estado EC2 `running` | Verificado |
| Asociación a subred de aplicación | Verificada |
| Asociación del Security Group | Verificada |
| IMDSv2 obligatorio | Verificado |
| Detección de volumen sin cifrar | Confirmada |
| Activación del cifrado EBS predeterminado | Verificada |
| Creación de nuevo volumen cifrado | Verificada |
| Creación de snapshot original | Verificada |
| Creación de copia cifrada | Verificada |
| Creación de volumen desde snapshot cifrado | No completada |
| Pruebas de conectividad real a EC2 | No realizadas |
| Verificación de cifrado físico | No realizada |

## 12. Relación con la ciberseguridad

Este laboratorio permitió aplicar y estudiar cuatro controles:

**Mínima exposición de red:** utilizar un Security Group que limita los puertos de entrada.

**Protección de metadatos:** configurar IMDSv2 obligatorio para reducir riesgos asociados al acceso a metadatos.

**Cifrado en reposo:** habilitar el cifrado EBS predeterminado para futuros volúmenes.

**Verificación y documentación:** comprobar los resultados mediante AWS CLI e identificar las limitaciones del entorno simulado.

Estos controles forman parte de una estrategia de defensa en profundidad y deben complementarse con IAM, actualizaciones de seguridad, monitoreo y otras medidas.

## 13. Conclusión

Se configuró una instancia EC2 simulada utilizando una AMI de Amazon Linux 2023, una subred de aplicación y un Security Group previamente definido.

También se verificó la configuración obligatoria de IMDSv2 y se identificó un volumen EBS registrado sin cifrado.

Como medida preventiva, se habilitó el cifrado EBS predeterminado y se comprobó que un nuevo volumen era registrado como cifrado.

Finalmente, se exploró una migración mediante snapshots. Aunque fue posible crear y consultar una copia cifrada, la creación de un volumen a partir de ella no se completó debido a una inconsistencia del emulador.

**El laboratorio demuestra la configuración y verificación de controles de seguridad mediante las API simuladas, sin afirmar que se ejecutaron máquinas virtuales reales, se cifraron físicamente los datos o se comprobó aislamiento de tráfico.**

La instancia original permanece registrada con su volumen principal sin cifrar, y este riesgo queda documentado como pendiente de remediación.