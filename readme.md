# 📦 Deploy Completo de WordPress com Docker na AWS 🚀

Este projeto demonstra uma arquitetura escalável de **WordPress** usando serviços da AWS como **EC2, RDS, EFS, Load Balancer (ALB)** e **Auto Scaling**, tudo orquestrado com **Docker e Docker Compose**.

---

## 🧱 Tecnologias e Serviços Utilizados

- **Amazon EC2 (privadas)** — para executar containers WordPress
- **Amazon RDS (MySQL)** — banco de dados gerenciado e persistente
- **Amazon EFS** — armazenamento compartilhado entre instâncias
- **Application Load Balancer (ALB)** — balanceamento de carga e entrada pública
- **Auto Scaling Group** — escalabilidade automática de instâncias EC2
- **Docker e Docker Compose** — orquestração de containers
- **Security Groups & VPC** — isolamento e controle de acesso em rede
- **NAT Gateway** — para acesso à internet em sub-redes privadas

---

## ⚙️ Funcionalidades e Objetivos

- Deploy automatizado do WordPress via **User Data**
- Instâncias EC2 **sem IP público** acessadas via **ALB**
- Montagem de volume compartilhado via **EFS** para persistência
- Banco de dados centralizado com RDS MySQL
- Escalabilidade com Auto Scaling

---

**Autor:** Rafael Dias  
**Objetivo:** Consolidar conhecimentos de Docker + AWS com foco em arquitetura escalável e segura.

---