## 1. 🌐 VPC e Security Groups

### Criando a VPC
- Acesse **VPC > Your VPCs > Create VPC**
- Crie sub-redes públicas (para ALB e NAT Gateway) e privadas (para EC2 e EFS)

### 🔄 Por que usar NAT Gateway?
A EC2 está em uma **sub-rede privada**, portanto precisa de acesso à internet para baixar pacotes via **NAT Gateway**.

---

### Security Groups (SG) necessários:

#### 🔐 `EC2_SG`
- **Inbound**: 
- **Outbound**: 

#### 🔐 `Load_Balancer_SG`
- **Inbound**: 
- **Outbound**: 

#### 🔐 `RDS_SG`
- **Inbound**: 
- **Outbound**: 

#### 🔐 `EFS_SG`
- **Inbound**: 
- **Outbound**: 

---

## 2. 🗄️ Criando o EFS e o Banco de Dados (RDS)

### EFS
- Criar o EFS em **Amazon EFS > Create File System**
- Criar Mount Targets para **cada AZ usada pelas EC2**
- Associar o `EFS_SG` ao EFS

### RDS (MySQL)
- Criar o banco com:
  - **Engine**: MySQL
  - **Usuário**: `admin`
  - **Database name**: `wordpress`
- Associar o `RDS_SG`
- Criar em sub-rede privada

---

## 3. 🧾 Script User Data para EC2

```bash
#!/bin/bash
apt-get update -y
apt-get install -y docker.io nfs-common

systemctl enable docker
systemctl start docker

curl -L "https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

mkdir -p /mnt/efs
mount -t efs {SEU_EFS_ID}:/ /mnt/efs

mkdir -p /mnt/efs/wordpress
chown -R 33:33 /mnt/efs/wordpress

mkdir -p /project/wordpress
cd /project/wordpress

cat > docker-compose.yml <<EOF
services:
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: {DATABASE_HOST}
      WORDPRESS_DB_USER: {USER_DATABASE}
      WORDPRESS_DB_PASSWORD: {SENHA_DATABASE}
      WORDPRESS_DB_NAME: {NOME_DATABASE}
    volumes:
      - /mnt/efs/wordpress:/var/www/html
EOF

docker-compose up -d

```

---

## 4. 📦 Launch Template

- Acesse **EC2 > Launch Templates**
- Crie um novo template com:
  - AMI: Ubuntu 22.04
  - Tipo: t2.micro ou superior
  - Key pair (opcional para debug)
  - User Data: script acima
  - SG: `EC2_SG`
  - Sem IP público

---

## 5. 📈 Auto Scaling Group

- Crie um novo Auto Scaling Group com o Launch Template acima
- Selecione sub-redes **privadas**
- Conecte ao Target Group do Load Balancer
- Capacidade:
  - Mínima: 2
  - Máxima: 4
- Regras de escalonamento com base em **uso de CPU**

---

## 6. ⚖️ Load Balancer (ALB)

- Criar um **Application Load Balancer**
- Sub-redes públicas
- Listener na porta 80
- Target Group:
  - Tipo: instance
  - Health Check: `/`
  - Porta: 80
- Conecte ao Auto Scaling Group

---

## ✅ Acesso à Aplicação

- Use o **DNS público do ALB** para acessar:
```
http://<ALB-DNS>.elb.amazonaws.com
```

- O WordPress estará disponível e acessível com arquivos persistidos no EFS.

