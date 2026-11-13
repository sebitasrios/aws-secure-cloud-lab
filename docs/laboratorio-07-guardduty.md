# Laboratorio 07 — Detección de amenazas con AWS GuardDuty

## Objetivo

Configurar un detector GuardDuty y comprobar la generación y consulta de hallazgos de seguridad mediante AWS Emulator local.

## Entorno

- AWS Emulator mediante Docker.
- AWS CLI y PowerShell.
- Endpoint local: https://127.0.0.1:8085
- Región simulada: us-east-1.
- No se utilizaron recursos de AWS real.

## 1. Consulta inicial

Se ejecutó guardduty list-detectors.

Resultado inicial:

DetectorIds: []

Esto confirmó que no había detectores registrados.

## 2. Creación del detector

Se ejecutó guardduty create-detector con la opción --enable.

ID generado:

b86ea088ab27a379df7f635bb66d0001

## 3. Verificación del detector

Se ejecutó guardduty get-detector.

Resultados:

- Status: ENABLED
- CreatedAt: 2026-11-13T23:40:53.219Z
- UpdatedAt: 2026-11-13T23:40:53.219Z

El detector quedó habilitado en el emulador.

## 4. Consulta inicial de hallazgos

Se ejecutó guardduty list-findings.

Resultado:

FindingIds: []

Inicialmente no existían hallazgos.

## 5. Generación de hallazgos simulados

Se ejecutó guardduty create-sample-findings.

Posteriormente se consultaron nuevamente los hallazgos mediante list-findings.

Resultado: el emulador devolvió una lista extensa de identificadores de hallazgos simulados.

Ejemplos de identificadores:

- d13f87820ab0c51d57d1758b99f700b1
- 48714ed2d8cc299c65201212d74f00b0
- 0391879b0563b3a96fb0100d8ab200af

## 6. Resultados

| Prueba | Resultado |
|---|---|
| Consultar detectores existentes | Exitosa |
| Crear detector | Exitosa |
| Verificar estado ENABLED | Exitosa |
| Consultar hallazgos iniciales | Sin hallazgos |
| Generar hallazgos de ejemplo | Exitosa |
| Consultar hallazgos posteriores | Hallazgos encontrados |
| Detectar ataques reales | No evaluado |

## 7. Importancia para la ciberseguridad

GuardDuty es un servicio orientado a identificar actividades potencialmente maliciosas mediante el análisis de señales de seguridad.

Los hallazgos permiten priorizar investigaciones y apoyar la respuesta ante incidentes.

La generación de hallazgos de ejemplo permite familiarizarse con el funcionamiento de las API sin realizar ataques reales.

## 8. Limitaciones

Las pruebas se realizaron exclusivamente en AWS Emulator.

Los hallazgos fueron generados artificialmente mediante create-sample-findings.

No se ejecutaron ataques reales ni se demostró la detección automática de amenazas.

Tampoco se analizaron los detalles individuales de cada hallazgo.

## Conclusión

Se creó y habilitó un detector GuardDuty simulado.

Se comprobó que inicialmente no existían hallazgos y que el emulador permitió generar y consultar hallazgos de ejemplo.

El laboratorio demuestra el uso básico de las API de GuardDuty en un entorno controlado.
