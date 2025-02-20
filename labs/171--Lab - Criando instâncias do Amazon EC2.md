# Configuração de Instâncias EC2 na AWS  

Este documento descreve os passos para iniciar e configurar instâncias EC2 na AWS, utilizando o Console de Gerenciamento da AWS e a AWS CLI.

---

## **Tarefa 1: Iniciar uma instância EC2 pelo Console**  

Nesta tarefa, criaremos uma instância EC2 que atuará como um **Bastion Host**, permitindo o acesso seguro a outras instâncias privadas na AWS.

### **1. Criar uma instância Bastion Host**  
1. Acesse o **Console de Gerenciamento da AWS**.  
2. No campo de pesquisa, digite **EC2** e clique para abrir o serviço.  
3. Clique em **Executar Instância**.  

### **2. Configurar a Instância**  

#### **Definir Nome e Tags**  
- **Nome da instância:** `Bastion host`  
- A AWS criará automaticamente uma tag com a chave `Name` e o valor inserido.  

#### **Escolher a AMI**  
- **Imagem do sistema operacional:** `Amazon Linux 2 (HVM)`  
- Essa AMI contém pacotes essenciais e suporte para a AWS CLI.  

#### **Selecionar o Tipo de Instância**  
- **Tipo:** `t3.micro`  
- Possui 2 vCPUs e 1GB de memória RAM, sendo adequado para testes e baixa carga.  

#### **Configurar Par de Chaves**  
- **Escolher:** `Prosseguir sem um par de chaves (não recomendado)`, pois usaremos **EC2 Instance Connect** para acessar a máquina.  

#### **Configurar Rede**  
- **VPC:** `Lab VPC` (criada automaticamente pelo ambiente de laboratório).  
- **Sub-rede:** `Pública` (permitindo acesso externo).  
- **Atribuir IP público:** `Habilitado`.  

#### **Definir o Grupo de Segurança**  
- **Criar novo grupo:** `Bastion security group`.  
- **Descrição:** `Permit SSH connections`.  
- **Regra de entrada:**  
  - Tipo: `SSH`  
  - Protocolo: `TCP`  
  - Porta: `22`  
  - Origem: `My IP` (para restringir o acesso ao seu IP).  

#### **Configurar Armazenamento**  
- **Volume de inicialização:** `8 GiB`, usando Amazon EBS (Elastic Block Store).  

#### **Perfil IAM**  
- **Selecionar:** `Bastion-Role`.  
- Esse perfil permite que a instância acesse outros serviços da AWS com permissões específicas.  

### **3. Iniciar Instância**  
1. Revise todas as configurações na seção **Resumo**.  
2. Clique em **Executar Instância**.  
3. Após a criação, vá para **Visualizar todas as instâncias** e aguarde até que o status seja **running**.  

---

## **Tarefa 2: Conectar-se ao Bastion Host**  

Agora, vamos nos conectar à instância Bastion usando o **EC2 Instance Connect**.

1. No Console EC2, selecione a instância **Bastion host**.  
2. Clique em **Conectar-se**.  
3. Na aba **EC2 Instance Connect**, clique em **Conectar**.  

Agora estamos dentro da instância Bastion e podemos usá-la para administrar outras instâncias EC2.

---

## **Tarefa 3: Criar uma nova instância EC2 pela AWS CLI**  

Nesta tarefa, criaremos uma nova instância EC2 que atuará como **Servidor Web**, instalando automaticamente o Apache via script de dados do usuário.

### **1. Definir a Região e Obter o ID da AMI**  
A AMI do Amazon Linux 2 é atualizada constantemente, então usamos um comando para obter a versão mais recente:  

```bash
AZ=`curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone`
export AWS_DEFAULT_REGION=${AZ::-1}
AMI=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)
echo $AMI
```

2. Obter o ID da Sub-rede
```bash
SUBNET=$(aws ec2 describe-subnets --filters 'Name=tag:Name,Values=Public Subnet' --query Subnets[].SubnetId --output text)
echo $SUBNET
```
3. Obter o ID do Grupo de Segurança
```bash
SG=$(aws ec2 describe-security-groups --filters Name=group-name,Values=WebSecurityGroup --query SecurityGroups[].GroupId --output text)
echo $SG
```
4. Baixar o Script de Configuração (User Data)
```bash
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-1-23732/171-lab-JAWS-create-ec2/s3/UserData.txt
cat UserData.txt
```
5. Criar a Instância Web Server
```bash
INSTANCE=$(
aws ec2 run-instances \
--image-id $AMI \
--subnet-id $SUBNET \
--security-group-ids $SG \
--user-data file:///home/ec2-user/UserData.txt \
--instance-type t3.micro \
--tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Web Server}]' \
--query 'Instances[*].InstanceId' \
--output text
)
echo $INSTANCE
```
6. Verificar Status da Instância
```bash
aws ec2 describe-instances --instance-ids $INSTANCE --query 'Reservations[].Instances[].State.Name' --output text
```
7. Testar o Servidor Web
```bash
aws ec2 describe-instances --instance-ids $INSTANCE --query Reservations[].Instances[].PublicDnsName --output text
```
Copie o DNS retornado e cole em uma nova aba do navegador. A página web deve ser carregada, confirmando que o servidor está rodando corretamente.

## Conclusão
Criamos um Bastion Host via AWS Console.
Utilizamos a AWS CLI para iniciar e configurar uma nova instância Web Server.
O servidor Apache foi instalado e testado com sucesso. 🚀
