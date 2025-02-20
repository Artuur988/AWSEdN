# Configuração de Instâncias EC2 na AWS  

Este documento descreve os passos para iniciar e configurar instâncias EC2 na AWS, utilizando o Console de Gerenciamento da AWS e a AWS CLI.

## **Tarefa 1: Iniciar uma instância EC2 pelo Console**  

### **1. Criar uma instância Bastion Host**  
1. No Console AWS, pesquise e abra o serviço **EC2**.  
2. Clique em **Executar Instância**.  

### **2. Configurar a Instância**  
- **Nome:** `Bastion host`  
- **AMI:** `Amazon Linux 2 (HVM)`  
- **Tipo de instância:** `t3.micro`  
- **Par de chaves:** `Prosseguir sem um par de chaves`  
- **Rede:**  
  - VPC: `Lab VPC`  
  - Sub-rede: `Pública`  
  - IP público: `Habilitado`  
- **Grupo de Segurança:**  
  - Nome: `Bastion security group`  
  - Descrição: `Permit SSH connections`  
- **Armazenamento:** `Padrão (8 GiB)`  
- **Perfil IAM:** `Bastion-Role`  

### **3. Iniciar Instância**  
1. Clique em **Executar Instância**.  
2. Escolha **Visualizar todas as instâncias**.  

---

## **Tarefa 2: Conectar-se ao Bastion Host**  
1. No Console EC2, selecione a instância Bastion.  
2. Clique em **Conectar-se** > **EC2 Instance Connect** > **Conectar**.  

---

## **Tarefa 3: Criar uma nova instância EC2 pela AWS CLI**  

### **1. Definir Região e Obter ID da AMI**  
```bash
AZ=`curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone`
export AWS_DEFAULT_REGION=${AZ::-1}
AMI=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)
echo $AMI
