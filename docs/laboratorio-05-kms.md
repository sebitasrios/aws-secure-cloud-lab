# Laboratorio 05 — Gestión de claves y cifrado con AWS KMS

## Objetivo

Evaluar la creación y utilización de claves criptográficas mediante AWS KMS, utilizando exclusivamente AWS Emulator local.

## Entorno

- AWS Emulator ejecutado mediante Docker.
- AWS CLI y PowerShell.
- Endpoint local: https://127.0.0.1:8085
- Región simulada: us-east-1.
- No se utilizaron servicios de AWS real.

## 1. Creación de la clave KMS

Se consultaron inicialmente las claves existentes.

Resultado: no había claves registradas.

Posteriormente se creó una clave KMS.

| Propiedad | Resultado |
|---|---|
| ID | 41bf327c-7772-4413-9483-c2660403d43f |
| Estado | Enabled |
| Habilitada | true |
| Región | us-east-1 |

## 2. Prueba de cifrado

Se creó el archivo `mensaje-kms.txt` con el contenido:

`Mensaje confidencial - AWS Secure Cloud Lab`

Se ejecutó la operación `kms encrypt` utilizando la clave creada.

Resultado: el emulador devolvió un valor codificado en Base64 correspondiente al resultado de cifrado.

El resultado se almacenó en los archivos locales `mensaje-cifrado.b64` y `mensaje-cifrado.bin`.

## 3. Prueba de descifrado

Se ejecutó `kms decrypt` utilizando el archivo binario generado.

Resultado recuperado:

`Mensaje confidencial - AWS Secure Cloud Lab`

La coincidencia con el mensaje original confirmó que las operaciones de cifrado y descifrado funcionaron dentro del emulador.

## 4. Resultados

| Prueba | Estado |
|---|---|
| Consultar claves KMS | Exitosa |
| Crear clave | Exitosa |
| Verificar estado Enabled | Exitosa |
| Cifrar mensaje | Exitosa |
| Generar archivo cifrado | Exitosa |
| Descifrar mensaje | Exitosa |
| Recuperar mensaje original | Exitosa |
| Restricción de permisos sobre la clave | No evaluada |

## 5. Importancia para la ciberseguridad

AWS KMS permite administrar claves utilizadas para proteger información.

El cifrado ayuda a mantener la confidencialidad de los datos. Sin embargo, también es necesario controlar qué identidades tienen permisos para utilizar las claves.

Por ello, el cifrado debe complementarse con políticas IAM, auditoría y controles de acceso.

## 6. Limitaciones

Las pruebas se realizaron exclusivamente mediante las API de AWS Emulator.

Los resultados verifican el comportamiento observado en el emulador, pero no demuestran protección criptográfica física ni equivalencia completa con AWS KMS real.

No se realizaron pruebas de políticas de acceso ni de denegación de permisos.

## Conclusión

Se creó una clave KMS simulada y se realizaron correctamente las operaciones de cifrado y descifrado.

El laboratorio permitió verificar el funcionamiento básico de KMS y comprender su relación con la protección de información en la nube.
