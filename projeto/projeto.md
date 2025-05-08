## 1. 🌐 VPC e Security Groups

### Criando a VPC
- Acesse **VPC > Your VPCs > Create VPC**
- Crie sub-redes públicas (para ALB e NAT Gateway) e privadas (para EC2 e EFS)
![VPC1](https://github.com/user-attachments/assets/fcdb1615-4968-4e64-bc7d-5572d7a5820a)
![VPC2](https://github.com/user-attachments/assets/27bca97e-06d0-494c-87a0-0056da0c6a7d)
![VPC3](https://github.com/user-attachments/assets/12082784-b3ca-495e-96c3-839e17f08ea5)

### 🔄 Por que usar NAT Gateway?
A EC2 está em uma **sub-rede privada**, portanto precisa de acesso à internet para baixar pacotes via **NAT Gateway**.

---

### Security Groups (SG) necessários:
Realize a criação dos SGs a seguir e delete todas as regras neles:

Ec2_SG,
EFS_SG,
RDS_SG,
Load_Balancer_SG 

Agora podemos definir as regras de cada grupo: 
#### 🔐 `EC2_SG`
- **Inbound**:ADICIONE SSH E para o SEU IP, ADICIONE HTTP e selecione o SG do Load Balancer, ADCIONE NFS e selecione o SG do EFS
- ![EC2_INBOUND](https://github.com/user-attachments/assets/ca5ae310-593a-4efb-a588-3a5a02f7166f)

- **Outbound**: 
![EC2_OUTBOUND](https://github.com/user-attachments/assets/452467bd-fb3f-4983-b7cc-6b60e59c6cb6)

#### 🔐 `Load_Balancer_SG`
- **Inbound**: Adicione HTTP e coloque para qualquer ipv4
- **Outbound**: Adicione HTTP e selecione o SG da EC2
![LB_SG](https://github.com/user-attachments/assets/17f45930-56f4-4d01-933b-14571c880902)

#### 🔐 `RDS_SG`
- **Inbound**: Adicione o MYSQL/Aurora e selecione o SG da EC2

- **Outbound**: Adicione o MYSQL/Aurora e selecione o SG da EC2
![RDS_SG](https://github.com/user-attachments/assets/362b15cd-0cc7-4baf-ae7c-dfa2a71e0328)

#### 🔐 `EFS_SG`
- **Inbound**:
- ![EFSG_INBOUND](https://github.com/user-attachments/assets/ef5e3b04-45ea-4428-9ea0-bce287c88611)

- **Outbound**: 
![EFS_OUTBOUND](https://github.com/user-attachments/assets/3401ed7e-9583-4687-b53e-c23dfde81245)

---

## 2. 🗄️ Criando o EFS e o Banco de Dados (RDS)

### EFS
- Criar o EFS em **Amazon EFS > Create File System**
- Criar Mount Targets para **cada AZ usada pelas EC2**
- Associar o `EFS_SG` ao EFS
![EFS1](https://github.com/user-attachments/assets/c4be35f2-6f0c-4c5c-9542-3a9b705f86b9)
![EFS2](https://github.com/user-attachments/assets/201d9464-c41b-47c6-8b21-0b7ef55fd5fc)
![EFS3](https://github.com/user-attachments/assets/7f661d93-58f7-45ed-b39c-bddd2f477355)
![EFS4](https://github.com/user-attachments/assets/2dd0e692-58dd-478f-a731-d8d5cd87289d)
![EFS5](https://github.com/user-attachments/assets/ffaf655b-4728-4a4b-bd34-b507733dd31a)
![EFS6](https://github.com/user-attachments/assets/8891033a-867a-43a2-96de-b3721a1e43c8)

### RDS (MySQL)

- Criar o banco com:
  - **Engine**: MySQL
  - **Usuário**: `admin`
  - **Database name**: `wordpress`
- Associar o `RDS_SG`
- Criar em sub-rede privada
Siga as instruções abaixo: 
![RDS1](https://github.com/user-attachments/assets/601cebbb-f175-4660-b26a-d61c060e2750)
![RDS2](https://github.com/user-attachments/assets/039118d2-a79c-475c-ba65-40f77bdff2ad)
![RDS3](https://github.com/user-attachments/assets/a6758dba-6676-433d-b2cf-40e0f860a654)
![RDS4](https://github.com/user-attachments/assets/50786562-a3f0-4dfb-b30b-ec244ec52af0)
![RDS5](https://github.com/user-attachments/assets/60a24e08-526d-4edf-84d2-1f527032e730)
![RDS6](https://github.com/user-attachments/assets/d2c21262-9aa0-4a62-a7eb-7a269830cfcb)
![RDS7](https://github.com/user-attachments/assets/7c21f9b5-ede8-463b-86d6-c2cfe9ac2183)

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
![Captura de tela de 2025-05-07 20-24-11](https://github.com/user-attachments/assets/37a6bbbb-d408-4742-9d5e-38db935d3556)

![Captura de tela de 2025-05-07 20-26-31](https://github.com/user-attachments/assets/43de6a95-b4e3-4219-92dd-cdcbdca9e3e4)

![Captura de tela de 2025-05-07 20-27-06](https://github.com/user-attachments/assets/c8dade79-7625-46b4-a5b1-551edffc2270)
![Captura de tela de 2025-05-07 20-29-48](https://github.com/user-attachments/assets/2a770ab1-a7e4-4c19-8c0e-f7623d8a9ea6)
![Captura de tela de 2025-05-07 21-01-54](https://github.com/user-attachments/assets/dd6cd94f-efc6-4ebe-8231-79123326bdc6)

---

## 5. ⚖️ Load Balancer (ALB)

- Criar um **Application Load Balancer**
- Sub-redes públicas
- Listener na porta 80
- Target Group:
  - Tipo: instance
  - Health Check: `/`
  - Porta: 80
- Conecte ao Auto Scaling Group
![Captura de tela de 2025-05-07 20-40-51](https://github.com/user-attachments/assets/02295efb-e4fa-4996-ba4d-e4e191641d9f)
![Captura de tela de 2025-05-07 20-41-42](https://github.com/user-attachments/assets/35232acc-542d-4882-88a4-78e59f4eee5e)
![Captura de tela de 2025-05-07 20-44-45](https://github.com/user-attachments/assets/7f5712b3-bf9b-4442-8ee4-1471a762e4ed)

---
## 6. 📈 Auto Scaling Group

- Crie um novo Auto Scaling Group com o Launch Template acima
- Selecione sub-redes **privadas**
- Conecte ao Target Group do Load Balancer
- Capacidade:
  - Mínima: 2
  - Máxima: 4
![Captura de tela de 2025-05-07 20-45-12](https://github.com/user-attachments/assets/de9bbdbf-232d-4e79-8f73-77a8576ce41d)
![Captura de tela de 2025-05-07 20-46-15](https://github.com/user-attachments/assets/ae5b2eba-8327-41f5-9fcc-62d9191deea1)
![Captura de tela de 2025-05-07 20-47-52](https://github.com/user-attachments/assets/0d6954c8-9794-4bc0-b226-7d28bdeedd6e)
![Captura de tela de 2025-05-07 20-49-01](https://github.com/user-attachments/assets/a5b928ac-f5f6-4808-9dc7-be6346a88d26)
![Captura de tela de 2025-05-07 20-49-49](https://github.com/user-attachments/assets/76c4af0d-1393-4f00-8b9c-91ba9d02afb6)
![Captura de tela de 2025-05-07 20-49-57](https://github.com/user-attachments/assets/b0830af5-f40b-4e2d-b8f6-df6e6ed7369f)
![Captura de tela de 2025-05-07 21-08-23](https://github.com/user-attachments/assets/5ee81d6e-f68e-49e9-8e8c-3f05f2b78f63)


---

## ✅ Acesso à Aplicação

- Use o **DNS público do ALB** para acessar:
```
http://<ALB-DNS>.elb.amazonaws.com
```

- O WordPress estará disponível e acessível com arquivos persistidos no EFS.

