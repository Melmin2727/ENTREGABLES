# Automatización del Despliegue con Terraform (Propuesta Conceptual)

## 1. Introducción

Si bien el despliegue actual del proyecto se realizó de forma manual/guiada mediante la consola de AWS y AWS CLI, este documento explica **conceptualmente** cómo podría automatizarse completamente la infraestructura de Comercial Nova utilizando **Terraform**, como mejora futura del proyecto (Infraestructura como Código — IaC).

No se incluye código completo de Terraform; el objetivo es documentar el enfoque, la estructura y los beneficios de esta automatización.

## 2. ¿Qué es Terraform y por qué se propone?

Terraform es una herramienta de Infraestructura como Código (IaC) desarrollada por HashiCorp que permite definir, versionar y desplegar infraestructura cloud mediante archivos declarativos (`.tf`), en lugar de crear los recursos manualmente a través de la consola web.

Su adopción para Comercial Nova aportaría:

- **Reproducibilidad**: la infraestructura completa (VPC, EC2, RDS, S3, ALB, Auto Scaling, IAM, CloudWatch) podría recrearse de forma idéntica en minutos, útil ante desastres o para crear ambientes de pruebas.
- **Versionado**: los cambios de infraestructura quedarían registrados en el repositorio Git, con historial y posibilidad de revertir cambios.
- **Consistencia entre ambientes**: se podrían definir ambientes de *desarrollo*, *pruebas* y *producción* con la misma base de código, variando solo parámetros.
- **Reducción de errores humanos**: se elimina la configuración manual repetitiva y propensa a errores en la consola.

## 3. Estructura de Directorios Propuesta

```
terraform/
├── main.tf              # Orquestación general de módulos
├── variables.tf          # Variables de entrada (región, CIDR, tipos de instancia, etc.)
├── outputs.tf             # Salidas relevantes (DNS del ALB, endpoint de RDS)
├── providers.tf           # Configuración del proveedor AWS
├── modules/
│   ├── vpc/                # Módulo de red: VPC, subredes, IGW, route tables
│   ├── security-groups/    # Módulo de Security Groups
│   ├── ec2-asg/              # Módulo de Launch Template + Auto Scaling Group
│   ├── alb/                  # Módulo de Application Load Balancer
│   ├── rds/                  # Módulo de base de datos RDS MySQL
│   ├── s3/                    # Módulo de bucket S3 (medios y respaldos)
│   ├── iam/                    # Módulo de roles y políticas IAM
│   └── cloudwatch/              # Módulo de alarmas y dashboards
└── environments/
    ├── dev.tfvars
    └── prod.tfvars
```

## 4. Enfoque por Módulos

Cada componente de la arquitectura se modelaría como un **módulo Terraform independiente y reutilizable**:

- **Módulo `vpc`**: definiría la VPC, las subredes públicas y privadas en múltiples AZ, el Internet Gateway y las tablas de rutas, replicando exactamente el diseño documentado en [`../../arquitectura/justificacion_arquitectura.md`](../../arquitectura/justificacion_arquitectura.md).
- **Módulo `security-groups`**: centralizaría las reglas de entrada/salida para el ALB, las instancias EC2 y RDS, evitando reglas abiertas innecesarias.
- **Módulo `rds`**: aprovisionaría la instancia MySQL con parámetros de backup, cifrado y el DB Subnet Group correspondiente.
- **Módulo `ec2-asg`**: definiría el Launch Template (basado en la AMI generada tras la configuración de WordPress) y el Auto Scaling Group con sus políticas de escalado.
- **Módulo `alb`**: crearía el Load Balancer, el Target Group y los Listeners.
- **Módulo `s3`**: crearía el bucket de medios/respaldos, con versionado y políticas de ciclo de vida.
- **Módulo `iam`**: definiría los roles e Instance Profiles necesarios para que EC2 acceda a S3 y CloudWatch sin credenciales estáticas.
- **Módulo `cloudwatch`**: centralizaría las alarmas y el dashboard de métricas.

## 5. Flujo de Trabajo Propuesto

1. `terraform init` — inicializa el directorio de trabajo y descarga el proveedor de AWS.
2. `terraform plan -var-file="environments/prod.tfvars"` — genera un plan de ejecución mostrando los recursos a crear/modificar/eliminar, sin aplicar cambios.
3. `terraform apply -var-file="environments/prod.tfvars"` — aplica los cambios planificados, creando la infraestructura completa.
4. `terraform destroy` — permite eliminar toda la infraestructura de forma controlada (útil en ambientes de pruebas académicas para evitar costos innecesarios).

## 6. Gestión del Estado (State)

Se recomendaría almacenar el archivo de estado de Terraform (`terraform.tfstate`) de forma remota y segura, utilizando:

- Un **bucket S3** dedicado (distinto al de medios) para almacenar el estado.
- Una tabla de **DynamoDB** para el bloqueo de estado (*state locking*), evitando aplicaciones concurrentes que corrompan el estado de la infraestructura.

## 7. Integración con CI/CD (Mejora Futura Complementaria)

La automatización con Terraform se integraría naturalmente con un pipeline de **AWS CodePipeline** o **GitHub Actions**, donde cada cambio en el código de infraestructura (Pull Request) dispararía automáticamente un `terraform plan`, y su fusión a la rama principal ejecutaría el `terraform apply`, logrando un flujo de **GitOps** para la infraestructura de Comercial Nova.

## 8. Beneficios Esperados de esta Automatización

| Beneficio | Descripción |
|---|---|
| Reducción de tiempo de despliegue | De horas (proceso manual) a minutos (`terraform apply`) |
| Trazabilidad | Todo cambio de infraestructura queda registrado en el control de versiones |
| Menor error humano | Se elimina la configuración manual repetitiva en la consola |
| Recuperación ante desastres | Posibilidad de recrear toda la infraestructura en una región alterna rápidamente |
| Ambientes replicables | Un mismo código base para *dev*, *staging* y *producción* |

## 9. Conclusión

Si bien el despliegue actual demostró de forma práctica los conceptos de virtualización y arquitectura en AWS mediante configuración manual y AWS CLI, la adopción de Terraform representa el siguiente paso natural de madurez del proyecto hacia un modelo de **Infraestructura como Código**, alineado con las prácticas DevOps modernas y con el pilar de **Excelencia Operacional** del AWS Well-Architected Framework.
