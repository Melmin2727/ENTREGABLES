# Alertas y Monitoreo Configurado — Amazon CloudWatch

## 1. Introducción

Este documento detalla las métricas y alarmas configuradas en Amazon CloudWatch para garantizar la observabilidad operativa de la infraestructura de Comercial Nova, permitiendo detectar y reaccionar ante condiciones anómalas antes de que impacten a los usuarios finales.

## 2. Métricas Monitoreadas

| Métrica | Servicio | Namespace CloudWatch | Descripción |
|---|---|---|---|
| CPUUtilization | EC2 / Auto Scaling Group | `AWS/EC2` | Porcentaje de uso de CPU por instancia y promedio del ASG |
| StatusCheckFailed | EC2 | `AWS/EC2` | Verifica fallas de hardware/software a nivel de instancia y sistema |
| MemoryUtilization | EC2 (vía CloudWatch Agent) | `CWAgent` | Porcentaje de uso de memoria RAM (no disponible por defecto, requiere agente) |
| CPUUtilization (RDS) | RDS | `AWS/RDS` | Porcentaje de uso de CPU de la instancia de base de datos |
| FreeStorageSpace | RDS | `AWS/RDS` | Espacio de almacenamiento libre en la instancia RDS |
| DatabaseConnections | RDS | `AWS/RDS` | Número de conexiones activas a la base de datos |
| RequestCount / TargetResponseTime | ALB | `AWS/ApplicationELB` | Volumen de solicitudes y tiempo de respuesta del balanceador |
| HealthyHostCount / UnHealthyHostCount | ALB / Target Group | `AWS/ApplicationELB` | Número de instancias saludables/no saludables detrás del ALB |
| GroupDesiredCapacity / GroupInServiceInstances | Auto Scaling | `AWS/AutoScaling` | Capacidad deseada vs. instancias realmente en servicio |

## 3. Alarmas Configuradas

### 3.1 CPU de EC2 mayor al 80%

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name alarma-cpu-alta-wordpress \
  --metric-name CPUUtilization --namespace AWS/EC2 \
  --statistic Average --period 300 --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=AutoScalingGroupName,Value=asg-wordpress-nova \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:xxxxxxxxxxxx:alerta-comercial-nova
```

**Justificación:** una CPU sostenida por encima del 80% durante dos periodos consecutivos de 5 minutos indica saturación de la instancia y debería, además de notificar, disparar la política de escalado del Auto Scaling Group.

### 3.2 Estado de salud de instancias EC2 (Status Check Failed)

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name alarma-status-check-ec2 \
  --metric-name StatusCheckFailed --namespace AWS/EC2 \
  --statistic Maximum --period 60 --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 2 \
  --dimensions Name=AutoScalingGroupName,Value=asg-wordpress-nova \
  --alarm-actions arn:aws:sns:us-east-1:xxxxxxxxxxxx:alerta-comercial-nova
```

**Justificación:** detecta fallas a nivel de instancia o del sistema subyacente (hardware del host), permitiendo que Auto Scaling reemplace automáticamente la instancia afectada.

### 3.3 Uso de memoria (vía CloudWatch Agent)

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name alarma-memoria-alta-wordpress \
  --metric-name mem_used_percent --namespace CWAgent \
  --statistic Average --period 300 --threshold 85 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:xxxxxxxxxxxx:alerta-comercial-nova
```

**Justificación:** la métrica de memoria no está disponible por defecto en CloudWatch para instancias EC2; se instaló el **CloudWatch Agent** en la AMI base para publicar esta métrica personalizada, dado que WordPress con múltiples plugins puede generar consumo elevado de RAM.

### 3.4 CPU de RDS mayor al 75%

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name alarma-cpu-alta-rds \
  --metric-name CPUUtilization --namespace AWS/RDS \
  --statistic Average --period 300 --threshold 75 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=DBInstanceIdentifier,Value=wordpress-db-comercial-nova \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:xxxxxxxxxxxx:alerta-comercial-nova
```

**Justificación:** un umbral ligeramente menor que en EC2 (75% vs. 80%) se estableció porque la base de datos es un recurso crítico compartido por ambas instancias de aplicación; una saturación en RDS afecta a todo el sitio simultáneamente.

### 3.5 Espacio de almacenamiento libre en RDS

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name alarma-storage-bajo-rds \
  --metric-name FreeStorageSpace --namespace AWS/RDS \
  --statistic Average --period 300 --threshold 2147483648 \
  --comparison-operator LessThanThreshold \
  --dimensions Name=DBInstanceIdentifier,Value=wordpress-db-comercial-nova \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:xxxxxxxxxxxx:alerta-comercial-nova
```

**Justificación:** umbral fijado en 2 GiB (2147483648 bytes) libres; por debajo de este valor existe riesgo de que la base de datos entre en estado de solo lectura o falle al escribir nuevas transacciones.

## 4. Integración con Amazon SNS

Todas las alarmas publican notificaciones en un tópico de **Amazon SNS** (`alerta-comercial-nova`), suscrito por:

- Correo electrónico del equipo DevOps.
- (Opcional/mejora futura) Webhook hacia un canal de Slack o Microsoft Teams del equipo técnico.

```bash
aws sns create-topic --name alerta-comercial-nova

aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:xxxxxxxxxxxx:alerta-comercial-nova \
  --protocol email \
  --notification-endpoint devops@comercialnova.com
```

## 5. Acciones Ejecutadas ante Alarmas

| Alarma | Acción automática | Acción manual del equipo |
|---|---|---|
| CPU EC2 > 80% | Disparo de política de escalado (Auto Scaling agrega instancia) | Revisar causa del incremento de tráfico o carga |
| Status Check Failed | Auto Scaling reemplaza la instancia afectada | Investigar logs de la instancia terminada |
| Memoria > 85% | Notificación vía SNS | Revisar plugins de WordPress con consumo elevado |
| CPU RDS > 75% | Notificación vía SNS | Evaluar escalado vertical de la instancia RDS |
| Storage RDS bajo | Notificación vía SNS (urgente) | Ampliar almacenamiento o depurar datos/logs |

## 6. Dashboard de Métricas

Se configuró un dashboard centralizado en CloudWatch (ver captura en [`dashboard_metricas.png`](dashboard_metricas.png)) que consolida en una sola vista:

- CPU promedio del Auto Scaling Group.
- Número de instancias saludables/no saludables en el ALB.
- CPU y conexiones activas de RDS.
- Latencia promedio de respuesta del ALB.

```bash
aws cloudwatch put-dashboard \
  --dashboard-name dashboard-comercial-nova \
  --dashboard-body file://dashboard-config.json
```

## 7. Conclusión

La configuración de alarmas descrita permite un modelo de **monitoreo proactivo**, donde el equipo técnico de Comercial Nova es notificado ante condiciones de riesgo antes de que se traduzcan en una caída del servicio, cumpliendo con el pilar de **Excelencia Operacional** del AWS Well-Architected Framework.
