# Laboratório: Alta Disponibilidade com Auto Scaling e Elastic Load Balancer

Este repositório documenta a implementação de uma arquitetura web altamente disponível e escalável na AWS, utilizando **Launch Templates**, **Auto Scaling Groups** e um **Application Load Balancer**. O projeto foi realizado como parte dos estudos para a certificação **AWS Certified Solutions Architect - Associate** na Escola da Nuvem.

O objetivo é demonstrar na prática como garantir que uma aplicação possa lidar com variações de tráfego e se recuperar automaticamente de falhas de instâncias.

**Instrutor:** João Gaioso ([@Gaiosojoao](https://github.com/Gaiosojoao))
**Aluno:** Artur Costa ([@arturcosta86](https://github.com/arturcosta86))

---

## 🎯 Objetivos do Laboratório

* **Criar um Launch Template:** Definir um modelo de configuração padronizado para as instâncias EC2, incluindo a AMI, o tipo de instância e um script de inicialização (`user data`).
* **Configurar um Application Load Balancer (ALB):** Distribuir o tráfego de entrada de forma inteligente entre múltiplas instâncias em diferentes Zonas de Disponibilidade.
* **Implementar um Auto Scaling Group (ASG):** Automatizar o provisionamento e o gerenciamento do ciclo de vida das instâncias EC2, mantendo a saúde e a quantidade desejada de servidores.
* **Testar a Alta Disponibilidade e o Balanceamento de Carga:** Validar que o tráfego está sendo distribuído entre as instâncias e que o sistema é resiliente. 

---

## 🛠️ Arquitetura da Solução

A arquitetura implementada é um padrão clássico para aplicações web na AWS, garantindo resiliência e escalabilidade.

O diagrama abaixo ilustra o fluxo:

```
                               +---------------------------+
                               | Application Load Balancer |
                               |      (Distributes        |
                               |        Traffic)           |
                               +-------------+-------------+
                                             |
                      +----------------------+----------------------+
                      |                                             |
                      V                                             V
            +-------------------+                           +-------------------+
            |  Availability     |                           |  Availability     |
            |     Zone A        |                           |     Zone B        |
            +-------------------+                           +-------------------+
                      |                                             |
            +---------+---------+                       +---------+---------+
            |  EC2 Instance 1   |                       |  EC2 Instance 2   |
            | (from Launch      |                       | (from Launch      |
            |    Template)      |                       |    Template)      |
            +-------------------+                       +-------------------+
                      ^                                             ^
                      |                                             |
                      +---------------------------------------------+
                                             |
                                 +-------------------------+
                                 |   Auto Scaling Group    |
                                 | (Maintains & Replaces   |
                                 |       Instances)        |
                                 +-------------------------+

```

**Fluxo Detalhado:**

1.  O **Launch Template** define como cada instância EC2 deve ser criada, incluindo o script `user data` que instala e inicia um servidor web Apache.
2.  O **Auto Scaling Group** utiliza o Launch Template para criar e manter um número desejado de instâncias (neste caso, 2) distribuídas em múltiplas Zonas de Disponibilidade (para alta disponibilidade). Se uma instância falhar, o ASG a substitui automaticamente.
3.  O **Application Load Balancer** recebe todo o tráfego do usuário e o distribui entre as instâncias saudáveis registradas em seu Target Group.

---

## 📄 Script de User Data (Código)

Este script é inserido no Launch Template e é executado automaticamente quando cada instância EC2 é iniciada. Ele instala um servidor web Apache e cria uma página `index.html` simples que exibe o endereço IP privado da própria instância, facilitando a visualização do balanceamento de carga.

```bash
#!/bin/bash
# Update the instance
yum update -y

# Install Apache Web Server (httpd)
yum install -y httpd

# Start the Apache service
systemctl start httpd

# Enable Apache to start on boot
systemctl enable httpd

# Create a simple HTML file to display the instance's private IP
echo "<h1>Servidor Web - Instância: $(hostname -f)</h1>" > /var/www/html/index.html
```

---

## ✅ Evidências da Implementação

As capturas de tela abaixo comprovam o funcionamento do balanceamento de carga. Ao acessar o DNS do Application Load Balancer repetidamente, o tráfego é direcionado para instâncias diferentes, cada uma exibindo seu próprio endereço IP interno.

### 1. Requisição Atendida pelo Servidor 1
* **Descrição:** A primeira atualização da página foi direcionada pelo Load Balancer para a primeira instância EC2 do grupo.

![Requisição no Servidor 1](https://github.com/arturcosta86/aws-ha-scaling-lab/blob/main/Evid%C3%AAncia%20-%20lab%20-%20Load%20Balancer%20e%20Auto%20Scaling%20-%20servidor%201.png)

### 2. Requisição Atendida pelo Servidor 2
* **Descrição:** Uma nova atualização da página foi direcionada para a segunda instância EC2, demonstrando que o balanceamento de carga está distribuindo o tráfego ativamente.

![Requisição no Servidor 2](https://github.com/arturcosta86/aws-ha-scaling-lab/blob/main/Evid%C3%AAncia%20-%20lab%20-%20Load%20Balancer%20e%20Auto%20Scaling%20-%20servidor%202.png)
