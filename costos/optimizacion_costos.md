# Estrategias de Optimización de Costos — Comercial Nova

## 1. Introducción

A partir de la estimación de costos documentada en [`estimacion_costos.md`](estimacion_costos.md), se identificaron diez estrategias técnicas para reducir el gasto operativo de la infraestructura de Comercial Nova sin comprometer la disponibilidad ni el desempeño del sitio WordPress.

## 2. Estrategias de Optimización

### 2.1 Uso de Instancias Reservadas o Savings Plans

Para las instancias EC2 y RDS que se mantienen en operación constante (carga base predecible), contratar **Reserved Instances** (1 o 3 años) o **Compute Savings Plans** puede reducir el costo de cómputo hasta en un **40-60%** respecto al precio On-Demand, dado que la carga mínima del Auto Scaling Group (2 instancias) es predecible y constante.

### 2.2 Uso de Instancias Spot para cargas no críticas

Para ambientes de prueba o staging (no productivos), se pueden utilizar **instancias Spot**, con descuentos de hasta el 90% respecto a On-Demand, aceptando el riesgo de interrupción, adecuado para entornos que no requieren disponibilidad continua.

### 2.3 Right-Sizing de instancias

Monitorear periódicamente las métricas de CPU y memoria en CloudWatch para verificar si el tipo de instancia (`t3.micro`) sigue siendo adecuado, evitando el sobreaprovisionamiento. Si la utilización promedio se mantiene consistentemente baja (<20%), se podría evaluar un tipo de instancia aún más pequeño o ajustar la capacidad mínima del Auto Scaling Group.

### 2.4 Políticas de ciclo de vida en S3 (Lifecycle Policies)

Configurar reglas de transición automática de los respaldos antiguos desde **S3 Standard** hacia **S3 Standard-IA** (tras 30 días) y posteriormente hacia **S3 Glacier Deep Archive** (tras 180 días), reduciendo el costo de almacenamiento de respaldos históricos hasta en un 80%.

### 2.5 Optimización del Auto Scaling con políticas de escalado más ajustadas

Ajustar el umbral de la política de Target Tracking Scaling (actualmente en 70% de CPU) en función del comportamiento real observado, evitando que el Auto Scaling agregue instancias de forma prematura ante picos breves y no sostenidos de tráfico.

### 2.6 Programar apagado de ambientes no productivos

Si se llegan a crear ambientes adicionales de desarrollo o pruebas, programar su apagado automático fuera del horario laboral mediante **AWS Instance Scheduler** o reglas de EventBridge, evitando pagar por cómputo inactivo durante las noches y fines de semana.

### 2.7 Uso de RDS con almacenamiento gp3 en lugar de io1/io2

El uso de almacenamiento **gp3** (en lugar de tipos de mayor performance como io1/io2, innecesarios para la carga actual de WordPress) permite obtener un rendimiento base suficiente a un costo significativamente menor, con la posibilidad de ajustar IOPS de forma independiente al tamaño del volumen.

### 2.8 Compresión y optimización de medios antes de subir a S3

Implementar la compresión automática de imágenes (mediante un plugin de optimización de WordPress) antes de almacenarlas en S3, reduciendo tanto el volumen de almacenamiento como los costos de transferencia de datos hacia los usuarios finales.

### 2.9 Uso de Amazon CloudFront como capa de caché (mejora futura con impacto en costos)

Aunque está fuera del alcance actual del proyecto, incorporar CloudFront permitiría cachear contenido estático en el borde de la red, **reduciendo la carga sobre las instancias EC2** (permitiendo un menor número de instancias mínimas en el Auto Scaling Group) y **disminuyendo el costo de transferencia de datos saliente** desde el origen.

### 2.10 Revisión periódica de recursos huérfanos (Cost Hygiene)

Establecer una revisión mensual (apoyada en **AWS Cost Explorer** y el inventario documentado en [`../aws/inventario_recursos_aws.md`](../aws/inventario_recursos_aws.md)) para identificar y eliminar recursos no utilizados: volúmenes EBS sin instancia asociada, snapshots antiguos innecesarios, direcciones IP elásticas no asociadas, o Load Balancers sin tráfico, los cuales generan costos silenciosos si no se auditan regularmente.

## 3. Resumen de Ahorro Potencial Estimado

| Estrategia | Ahorro potencial estimado |
|---|---|
| Instancias Reservadas / Savings Plans (EC2 + RDS) | 30% - 50% del costo de cómputo |
| Lifecycle Policies en S3 | Hasta 80% del costo de almacenamiento de respaldos antiguos |
| Right-sizing de instancias | 10% - 20% del costo de cómputo |
| CloudFront como caché (futuro) | 15% - 25% del costo de transferencia y cómputo EC2 |
| Auditoría de recursos huérfanos | Variable, típicamente 5% - 10% del gasto total mensual |

## 4. Conclusión

La combinación de estas diez estrategias permitiría a Comercial Nova reducir el costo mensual estimado de infraestructura (documentado en [`estimacion_costos.md`](estimacion_costos.md)) en un rango aproximado del **20% al 40%** una vez maduren los patrones de uso reales de la plataforma, sin sacrificar la disponibilidad ni la escalabilidad diseñadas en la arquitectura, en línea con el pilar de **Optimización de Costos** del AWS Well-Architected Framework.
