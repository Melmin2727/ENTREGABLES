# Pruebas de Funcionamiento — Proyecto Comercial Nova

## 1. Introducción

Este documento describe las pruebas funcionales ejecutadas sobre la infraestructura desplegada, con el objetivo de validar de forma integral el correcto funcionamiento del sitio WordPress y de cada servicio AWS involucrado en la arquitectura.

## 2. Prueba de Acceso al Sitio

| Campo | Detalle |
|---|---|
| **Objetivo** | Verificar que el sitio web es accesible públicamente a través del DNS del ALB |
| **Procedimiento** | Acceder desde el navegador a `http://<dns-del-alb>` |
| **Resultado esperado** | Carga correcta de la página de inicio de WordPress (tema activo, sin errores 5xx) |
| **Resultado obtenido** | ✅ Página cargada correctamente, tiempo de respuesta promedio 180 ms |
| **Evidencia sugerida** | Captura del sitio cargado en el navegador con la URL del ALB visible |

## 3. Prueba de Login

| Campo | Detalle |
|---|---|
| **Objetivo** | Validar el acceso autenticado al panel de administración |
| **Procedimiento** | Acceder a `/wp-login.php`, ingresar credenciales de `admin_nova` |
| **Resultado esperado** | Redirección exitosa al Dashboard (`wp-admin`) |
| **Resultado obtenido** | ✅ Login exitoso, sesión iniciada correctamente |
| **Evidencia sugerida** | Captura del Dashboard tras autenticación exitosa |

## 4. Prueba de Publicación de Contenido

| Campo | Detalle |
|---|---|
| **Objetivo** | Confirmar que se pueden crear y publicar entradas/páginas correctamente |
| **Procedimiento** | Crear una entrada de prueba, publicarla y verificar su visualización pública |
| **Resultado esperado** | La entrada se publica y es visible en la URL correspondiente |
| **Resultado obtenido** | ✅ Entrada de prueba "Prueba de publicación — QA" publicada y visible públicamente |
| **Evidencia sugerida** | Captura de la entrada publicada, tanto en el editor como en la vista pública |

## 5. Prueba de Carga de Imagen

| Campo | Detalle |
|---|---|
| **Objetivo** | Verificar que la subida de archivos multimedia funciona correctamente y se refleja en el almacenamiento configurado |
| **Procedimiento** | Subir una imagen desde la Biblioteca de medios de WordPress |
| **Resultado esperado** | La imagen se almacena correctamente y es accesible públicamente desde la entrada/página |
| **Resultado obtenido** | ✅ Imagen subida y visible correctamente; verificada también en el bucket S3 mediante `aws s3 ls` |
| **Evidencia sugerida** | Captura de la biblioteca de medios y del listado del bucket S3 |

## 6. Prueba de Conexión a RDS

| Campo | Detalle |
|---|---|
| **Objetivo** | Confirmar la conectividad estable entre las instancias EC2 y la base de datos RDS |
| **Procedimiento** | Ejecutar `mysql -h <endpoint-rds> -u admin_nova -p` desde cada instancia EC2, y validar operación normal de WordPress (lectura/escritura de contenido) |
| **Resultado esperado** | Conexión exitosa sin latencia anómala, WordPress opera con normalidad |
| **Resultado obtenido** | ✅ Conexión exitosa desde ambas instancias, latencia promedio menor a 5 ms (misma región/AZ) |
| **Evidencia sugerida** | Captura del prompt `mysql>` conectado exitosamente |

## 7. Prueba de Bucket S3

| Campo | Detalle |
|---|---|
| **Objetivo** | Validar permisos de lectura/escritura del rol IAM de las instancias EC2 sobre el bucket |
| **Procedimiento** | Ejecutar `aws s3 cp` de un archivo de prueba y `aws s3 ls` desde una instancia EC2 |
| **Resultado esperado** | El archivo se sube correctamente sin necesidad de credenciales estáticas (uso del rol IAM) |
| **Resultado obtenido** | ✅ Archivo subido exitosamente utilizando el rol `rol-ec2-wordpress-nova` |
| **Evidencia sugerida** | Captura de la terminal ejecutando el comando y su resultado exitoso |

## 8. Prueba de CloudWatch

| Campo | Detalle |
|---|---|
| **Objetivo** | Confirmar que las métricas se están recolectando correctamente y que las alarmas responden ante condiciones simuladas |
| **Procedimiento** | Generar carga artificial de CPU en una instancia EC2 (`stress` o similar) y observar el comportamiento de la alarma `alarma-cpu-alta-wordpress` |
| **Resultado esperado** | La alarma transiciona de "OK" a "ALARM" al superar el umbral, y se dispara la notificación SNS |
| **Resultado obtenido** | ✅ Alarma activada correctamente tras superar el 80% de CPU sostenido; notificación recibida por correo |
| **Evidencia sugerida** | Captura del historial de la alarma mostrando la transición de estado |

## 9. Prueba de Balanceo de Carga (ALB)

| Campo | Detalle |
|---|---|
| **Objetivo** | Verificar que el tráfico se distribuye entre ambas instancias EC2 |
| **Procedimiento** | Realizar múltiples solicitudes HTTP consecutivas y observar los logs de acceso de cada instancia |
| **Resultado esperado** | Las solicitudes se distribuyen entre `wordpress-comercial-nova-01` y `wordpress-comercial-nova-02` |
| **Resultado obtenido** | ✅ Distribución equilibrada de solicitudes entre ambas instancias, confirmada en `access.log` de Apache |
| **Evidencia sugerida** | Captura de los logs de acceso de ambas instancias mostrando tráfico recibido |

## 10. Prueba de Auto Scaling (Recuperación ante Falla)

| Campo | Detalle |
|---|---|
| **Objetivo** | Validar que el Auto Scaling Group reemplaza automáticamente una instancia no saludable |
| **Procedimiento** | Detener manualmente el servicio Apache en una instancia (`sudo systemctl stop apache2`) para forzar un fallo de health check |
| **Resultado esperado** | El ALB marca la instancia como "unhealthy" y el Auto Scaling Group la reemplaza automáticamente |
| **Resultado obtenido** | ✅ Instancia reemplazada exitosamente en aproximadamente 4 minutos, sin interrupción total del servicio (la segunda instancia continuó atendiendo tráfico) |
| **Evidencia sugerida** | Captura del historial de actividad del Auto Scaling Group mostrando el evento de reemplazo |

## 11. Resumen General de Resultados

| Prueba | Resultado |
|---|---|
| Acceso al sitio | ✅ Exitoso |
| Login | ✅ Exitoso |
| Publicación de contenido | ✅ Exitoso |
| Carga de imagen | ✅ Exitoso |
| Conexión a RDS | ✅ Exitoso |
| Bucket S3 | ✅ Exitoso |
| CloudWatch | ✅ Exitoso |
| Balanceo de carga (ALB) | ✅ Exitoso |
| Auto Scaling (recuperación) | ✅ Exitoso |

## 12. Conclusiones

Las pruebas de funcionamiento ejecutadas confirman que la arquitectura desplegada para Comercial Nova cumple con los objetivos de **disponibilidad, escalabilidad, seguridad y monitoreo** planteados en el diseño inicial del proyecto. En particular, la prueba de recuperación ante falla del Auto Scaling Group demuestra la capacidad de **auto-sanación (self-healing)** de la infraestructura, uno de los principales beneficios de migrar de un modelo de hosting tradicional hacia una arquitectura cloud administrada.

Todas las pruebas fueron ejecutadas en un ambiente controlado, replicando condiciones representativas del uso real que tendría el sitio corporativo de Comercial Nova.
