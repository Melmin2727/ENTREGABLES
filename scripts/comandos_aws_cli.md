# Comandos AWS CLI Utilizados — Proyecto Comercial Nova

## 1. Introducción

Este documento reúne todos los comandos de AWS CLI empleados durante el ciclo de vida del proyecto: creación de red, cómputo, base de datos, almacenamiento, monitoreo y verificación del despliegue. Todos los comandos asumen que el perfil de AWS CLI ya fue configurado mediante `aws configure`.

## 2. Configuración Inicial

```bash
aws configure
aws sts get-caller-identity
```

## 3. Red — VPC, Subredes e Internet Gateway

```bash
# Crear la VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications \
  'ResourceType=vpc,Tags=[{Key=Name,Value=vpc-comercial-nova}]'

# Crear subredes públicas y privadas
aws ec2 create-subnet --vpc-id vpc-xxxxxxxx --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a --tag-specifications \
  'ResourceType=subnet,Tags=[{Key=Name,Value=subnet-publica-a}]'

aws ec2 create-subnet --vpc-id vpc-xxxxxxxx --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b --tag-specifications \
  'ResourceType=subnet,Tags=[{Key=Name,Value=subnet-publica-b}]'

aws ec2 create-subnet --vpc-id vpc-xxxxxxxx --cidr-block 10.0.11.0/24 \
  --availability-zone us-east-1a --tag-specifications \
  'ResourceType=subnet,Tags=[{Key=Name,Value=subnet-privada-app-a}]'

aws ec2 create-subnet --vpc-id vpc-xxxxxxxx --cidr-block 10.0.21.0/24 \
  --availability-zone us-east-1a --tag-specifications \
  'ResourceType=subnet,Tags=[{Key=Name,Value=subnet-privada-datos-a}]'

# Internet Gateway
aws ec2 create-internet-gateway --tag-specifications \
  'ResourceType=internet-gateway,Tags=[{Key=Name,Value=igw-comercial-nova}]'
aws ec2 attach-internet-gateway --vpc-id vpc-xxxxxxxx --internet-gateway-id igw-xxxxxxxx

# Tabla de rutas pública
aws ec2 create-route-table --vpc-id vpc-xxxxxxxx
aws ec2 create-route --route-table-id rtb-xxxxxxxx \
  --destination-cidr-block 0.0.0.0/0 --gateway-id igw-xxxxxxxx
aws ec2 associate-route-table --subnet-id subnet-xxxxxxxx --route-table-id rtb-xxxxxxxx
```

## 4. Security Groups

```bash
# SG del ALB
aws ec2 create-security-group --group-name sg-alb \
  --description "Trafico HTTP/HTTPS publico" --vpc-id vpc-xxxxxxxx

aws ec2 authorize-security-group-ingress --group-id sg-xxxxxxxx \
  --protocol tcp --port 80 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-id sg-xxxxxxxx \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# SG de EC2 (solo desde el ALB)
aws ec2 create-security-group --group-name sg-ec2-wordpress \
  --description "Trafico solo desde el ALB" --vpc-id vpc-xxxxxxxx

aws ec2 authorize-security-group-ingress --group-id sg-yyyyyyyy \
  --protocol tcp --port 80 --source-group sg-xxxxxxxx

# SG de RDS (solo desde EC2)
aws ec2 create-security-group --group-name sg-rds \
  --description "Trafico MySQL solo desde EC2" --vpc-id vpc-xxxxxxxx

aws ec2 authorize-security-group-ingress --group-id sg-zzzzzzzz \
  --protocol tcp --port 3306 --source-group sg-yyyyyyyy
```

## 5. EC2 — Instancias

```bash
# Lanzar instancia EC2
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name comercial-nova-key \
  --security-group-ids sg-yyyyyyyy \
  --subnet-id subnet-privada-app-a \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=wordpress-comercial-nova-01}]'

# Listar instancias
aws ec2 describe-instances --filters "Name=tag:Name,Values=wordpress-comercial-nova-*"

# Verificar estado de una instancia
aws ec2 describe-instance-status --instance-ids i-0123456789abcdef0

# Crear AMI a partir de la instancia configurada
aws ec2 create-image \
  --instance-id i-0123456789abcdef0 \
  --name "wordpress-comercial-nova-ami" \
  --no-reboot

# Detener / iniciar / terminar instancia (operaciones de mantenimiento)
aws ec2 stop-instances --instance-ids i-0123456789abcdef0
aws ec2 start-instances --instance-ids i-0123456789abcdef0
```

## 6. Amazon RDS

```bash
# Crear el DB Subnet Group
aws rds create-db-subnet-group \
  --db-subnet-group-name subnet-group-rds-nova \
  --db-subnet-group-description "Subredes privadas para RDS" \
  --subnet-ids subnet-privada-datos-a subnet-privada-datos-b

# Crear la instancia RDS MySQL
aws rds create-db-instance \
  --db-instance-identifier wordpress-db-comercial-nova \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --engine-version 8.0 \
  --master-username admin_nova \
  --master-user-password '********' \
  --allocated-storage 20 \
  --vpc-security-group-ids sg-zzzzzzzz \
  --db-subnet-group-name subnet-group-rds-nova \
  --no-publicly-accessible \
  --backup-retention-period 7

# Describir la instancia RDS (obtener endpoint)
aws rds describe-db-instances --db-instance-identifier wordpress-db-comercial-nova

# Crear snapshot manual
aws rds create-db-snapshot \
  --db-instance-identifier wordpress-db-comercial-nova \
  --db-snapshot-identifier wordpress-db-snapshot-manual-01
```

## 7. Amazon S3

```bash
# Crear el bucket de medios y respaldos
aws s3 mb s3://comercial-nova-media --region us-east-1

# Habilitar versionado
aws s3api put-bucket-versioning \
  --bucket comercial-nova-media \
  --versioning-configuration Status=Enabled

# Subir un respaldo de base de datos
aws s3 cp backup_wordpress_nova.sql s3://comercial-nova-media/backups/

# Listar contenido del bucket
aws s3 ls s3://comercial-nova-media/ --recursive

# Sincronizar carpeta de medios
aws s3 sync /var/www/html/wp-content/uploads/ s3://comercial-nova-media/uploads/

# Configurar política de ciclo de vida (mover respaldos antiguos a Glacier)
aws s3api put-bucket-lifecycle-configuration \
  --bucket comercial-nova-media \
  --lifecycle-configuration file://lifecycle-policy.json
```

## 8. Application Load Balancer y Target Group

```bash
# Crear el Target Group
aws elbv2 create-target-group \
  --name tg-wordpress-nova \
  --protocol HTTP --port 80 \
  --vpc-id vpc-xxxxxxxx \
  --health-check-path /wp-login.php \
  --target-type instance

# Crear el Application Load Balancer
aws elbv2 create-load-balancer \
  --name alb-comercial-nova \
  --subnets subnet-publica-a subnet-publica-b \
  --security-groups sg-xxxxxxxx \
  --scheme internet-facing \
  --type application

# Crear el Listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:...:loadbalancer/app/alb-comercial-nova/xxxx \
  --protocol HTTP --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...:targetgroup/tg-wordpress-nova/xxxx

# Describir el ALB (obtener DNS público)
aws elbv2 describe-load-balancers --names alb-comercial-nova
```

## 9. Auto Scaling Group

```bash
# Crear el Launch Template
aws ec2 create-launch-template \
  --launch-template-name lt-wordpress-nova \
  --version-description "v1" \
  --launch-template-data '{"ImageId":"ami-xxxxxxxxxxxxxxxxx","InstanceType":"t3.micro","SecurityGroupIds":["sg-yyyyyyyy"],"KeyName":"comercial-nova-key"}'

# Crear el Auto Scaling Group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name asg-wordpress-nova \
  --launch-template LaunchTemplateName=lt-wordpress-nova,Version='$Latest' \
  --min-size 2 --max-size 4 --desired-capacity 2 \
  --vpc-zone-identifier "subnet-privada-app-a,subnet-privada-app-b" \
  --target-group-arns arn:aws:elasticloadbalancing:...:targetgroup/tg-wordpress-nova/xxxx

# Describir el Auto Scaling Group
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names asg-wordpress-nova

# Crear política de escalado (basada en CPU)
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name asg-wordpress-nova \
  --policy-name escalado-cpu-nova \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{"PredefinedMetricSpecification":{"PredefinedMetricType":"ASGAverageCPUUtilization"},"TargetValue":70.0}'
```

## 10. Amazon CloudWatch

```bash
# Listar métricas disponibles
aws cloudwatch list-metrics --namespace AWS/EC2

# Crear una alarma de CPU alta
aws cloudwatch put-metric-alarm \
  --alarm-name alarma-cpu-alta-wordpress \
  --metric-name CPUUtilization --namespace AWS/EC2 \
  --statistic Average --period 300 --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=AutoScalingGroupName,Value=asg-wordpress-nova \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:xxxxxxxxxxxx:alerta-comercial-nova

# Describir alarmas configuradas
aws cloudwatch describe-alarms --alarm-name-prefix alarma-
```

## 11. IAM

```bash
# Crear rol para instancias EC2 (acceso a S3 y CloudWatch)
aws iam create-role \
  --role-name rol-ec2-wordpress-nova \
  --assume-role-policy-document file://trust-policy-ec2.json

aws iam attach-role-policy \
  --role-name rol-ec2-wordpress-nova \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

aws iam attach-role-policy \
  --role-name rol-ec2-wordpress-nova \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

# Crear Instance Profile y asociarlo
aws iam create-instance-profile --instance-profile-name perfil-ec2-wordpress-nova
aws iam add-role-to-instance-profile \
  --instance-profile-name perfil-ec2-wordpress-nova \
  --role-name rol-ec2-wordpress-nova
```

## 12. Comandos de Verificación General

```bash
aws ec2 describe-instances
aws s3 ls
aws rds describe-db-instances
aws autoscaling describe-auto-scaling-groups
aws cloudwatch list-metrics
aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:...:targetgroup/tg-wordpress-nova/xxxx
aws iam list-roles --query "Roles[?contains(RoleName,'nova')]"
```

## 13. Buenas Prácticas Aplicadas en el Uso de AWS CLI

- Uso de perfiles (`--profile`) separados para evitar operar accidentalmente sobre cuentas incorrectas.
- Uso de `--dry-run` en comandos críticos antes de ejecutar cambios definitivos.
- Uso de etiquetas (`Tags`) consistentes en todos los recursos para facilitar el inventario (ver [`../../aws/inventario_recursos_aws.md`](../../aws/inventario_recursos_aws.md)).
- Nunca se incluyeron credenciales ni contraseñas en texto plano dentro de scripts versionados en el repositorio.
