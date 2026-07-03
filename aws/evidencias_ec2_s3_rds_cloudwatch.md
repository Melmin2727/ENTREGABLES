# Evidencias de Servicios AWS — Comercial Nova

## 1. Introducción

Este documento detalla las capturas de pantalla necesarias para evidenciar el correcto funcionamiento de cada servicio AWS utilizado en el proyecto. Las capturas reales deben almacenarse en [`../evidencias/capturas_servicios/`](../evidencias/capturas_servicios/) siguiendo la convención de nombres indicada en cada sección.

## 2. Amazon VPC

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| Detalle de la VPC | Consola EC2 → VPC, mostrando CIDR `10.0.0.0/16` y estado "Available" | `vpc_detalle.png` |
| Listado de subredes | 6 subredes (2 públicas, 2 privadas app, 2 privadas datos) con sus CIDR y AZ | `vpc_subredes.png` |
| Internet Gateway asociado | Confirmación de que `igw-comercial-nova` está adjunto (`attached`) a la VPC | `vpc_igw.png` |
| Tablas de rutas | Ruta `0.0.0.0/0` hacia el IGW en la tabla pública | `vpc_route_tables.png` |

## 3. Subredes

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| Mapa de subredes por AZ | Vista de "Subnets" en consola, filtrado por VPC del proyecto | `subredes_por_az.png` |
| Auto-assign IP pública | Confirmación de que las subredes públicas tienen auto-asignación de IP habilitada y las privadas deshabilitada | `subredes_ip_publica.png` |

## 4. Internet Gateway

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| Estado del IGW | Estado "Attached" al `vpc-comercial-nova` | `internet_gateway.png` |

## 5. Amazon EC2

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| Listado de instancias | Ambas instancias WordPress en estado "Running", con su AZ y tipo `t3.micro` | `ec2_listado_instancias.png` |
| Detalle de instancia | Pestaña "Details" mostrando Security Group, subred e Instance Profile asociado | `ec2_detalle_instancia.png` |
| Status Checks | "2/2 checks passed" para ambas instancias | `ec2_status_checks.png` |
| Conexión SSH exitosa | Terminal mostrando conexión exitosa vía SSH/Session Manager | `ec2_conexion_ssh.png` |
| AMI generada | Consola de AMIs mostrando `wordpress-comercial-nova-ami` en estado "Available" | `ec2_ami_generada.png` |

## 6. Security Groups

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| Reglas de `sg-alb` | Entrada 80 y 443 desde `0.0.0.0/0` | `sg_alb_reglas.png` |
| Reglas de `sg-ec2-wordpress` | Entrada 80 solo desde `sg-alb`, 22 desde IP administrativa | `sg_ec2_reglas.png` |
| Reglas de `sg-rds` | Entrada 3306 solo desde `sg-ec2-wordpress` | `sg_rds_reglas.png` |

## 7. Amazon RDS

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| Instancia RDS disponible | Estado "Available", motor MySQL 8.0, clase `db.t3.micro` | `rds_instancia_disponible.png` |
| Endpoint de conexión | Endpoint y puerto 3306 visibles en el detalle de la instancia | `rds_endpoint.png` |
| Conectividad y seguridad | Confirmación de "Publicly Accessible: No" y SG asociado | `rds_conectividad.png` |
| Snapshot manual | Listado de snapshots, incluyendo el snapshot manual creado | `rds_snapshot.png` |
| Conexión exitosa desde EC2 | Prompt `mysql>` conectado desde una instancia EC2 | `rds_conexion_ec2.png` |

## 8. Amazon S3

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| Bucket creado | `comercial-nova-media` visible en el listado de buckets | `s3_bucket_creado.png` |
| Versionado habilitado | Pestaña "Properties" mostrando "Bucket Versioning: Enabled" | `s3_versionado.png` |
| Contenido del bucket | Carpetas `uploads/` y `backups/` con archivos cargados | `s3_contenido.png` |
| Cifrado en reposo | "Default encryption: SSE-S3" habilitado | `s3_cifrado.png` |

## 9. Amazon CloudWatch

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| Dashboard general | Vista completa de `dashboard-comercial-nova` (ver [`../monitoreo/dashboard_metricas.png`](../monitoreo/dashboard_metricas.png)) | `cloudwatch_dashboard.png` |
| Alarmas en estado OK | Listado de las 4 alarmas configuradas, todas en estado "OK" | `cloudwatch_alarmas.png` |
| Historial de una alarma | Gráfico histórico de `alarma-cpu-alta-wordpress` | `cloudwatch_historial_alarma.png` |
| Métricas de RDS | Gráfico de CPU y conexiones de la instancia RDS | `cloudwatch_metricas_rds.png` |

## 10. AWS IAM

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| Rol de EC2 | Detalle de `rol-ec2-wordpress-nova` con políticas adjuntas | `iam_rol_ec2.png` |
| Políticas adjuntas | `AmazonS3ReadOnlyAccess` y `CloudWatchAgentServerPolicy` visibles | `iam_politicas.png` |
| Instance Profile | Confirmación de asociación del perfil a las instancias EC2 | `iam_instance_profile.png` |

## 11. Application Load Balancer (ALB)

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| ALB activo | Estado "Active", esquema "internet-facing" | `alb_activo.png` |
| DNS del ALB | Nombre DNS público visible en el detalle | `alb_dns.png` |
| Target Group saludable | 2 instancias en estado "healthy" dentro de `tg-wordpress-nova` | `alb_target_group_healthy.png` |
| Listener configurado | Listener HTTP:80 reenviando al Target Group | `alb_listener.png` |

## 12. Auto Scaling Group

| Evidencia | Descripción | Archivo sugerido |
|---|---|---|
| ASG activo | `asg-wordpress-nova` con capacidad deseada 2, mínima 2, máxima 4 | `asg_activo.png` |
| Instancias gestionadas | 2 instancias EC2 bajo el control del ASG | `asg_instancias.png` |
| Política de escalado | Target Tracking al 70% de CPU configurada | `asg_politica_escalado.png` |
| Historial de actividad | Registro de eventos de escalado (si se generaron durante pruebas de carga) | `asg_historial.png` |

## 13. Resumen de Verificación

| Servicio | Verificado | Estado |
|---|---|---|
| VPC y subredes | ✅ | Correcto |
| Internet Gateway | ✅ | Correcto |
| Security Groups | ✅ | Correcto |
| EC2 | ✅ | Correcto |
| RDS | ✅ | Correcto |
| S3 | ✅ | Correcto |
| CloudWatch | ✅ | Correcto |
| IAM | ✅ | Correcto |
| ALB | ✅ | Correcto |
| Auto Scaling | ✅ | Correcto |

## 14. Conclusión

La evidencia recopilada confirma que todos los servicios AWS involucrados en la arquitectura de Comercial Nova se encuentran correctamente configurados, operativos y comunicados entre sí, validando el cumplimiento de los objetivos técnicos del proyecto.
