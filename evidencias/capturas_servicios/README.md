# Capturas de Servicios AWS

Esta carpeta almacena las capturas de pantalla reales de los servicios AWS desplegados para el proyecto Comercial Nova.

La lista completa de capturas requeridas, con su nombre de archivo sugerido y el propósito de cada una, está documentada en [`../../aws/evidencias_ec2_s3_rds_cloudwatch.md`](../../aws/evidencias_ec2_s3_rds_cloudwatch.md).

## Convención de nombres

Los archivos deben guardarse siguiendo el patrón `servicio_descripcion.png`, por ejemplo:

- `ec2_listado_instancias.png`
- `rds_instancia_disponible.png`
- `s3_bucket_creado.png`
- `cloudwatch_dashboard.png`
- `alb_target_group_healthy.png`
- `asg_activo.png`
- `iam_rol_ec2.png`
- `sg_ec2_reglas.png`
- `vpc_detalle.png`

## Checklist de capturas pendientes

- [ ] VPC y subredes
- [ ] Internet Gateway
- [ ] Security Groups (ALB, EC2, RDS)
- [ ] Instancias EC2 (listado, detalle, status checks)
- [ ] AMI generada
- [ ] Amazon RDS (instancia, endpoint, snapshot)
- [ ] Amazon S3 (bucket, versionado, contenido)
- [ ] Amazon CloudWatch (dashboard, alarmas)
- [ ] AWS IAM (rol, políticas, instance profile)
- [ ] Application Load Balancer (activo, DNS, target group)
- [ ] Auto Scaling Group (activo, política de escalado, historial)
