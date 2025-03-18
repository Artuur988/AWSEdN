# Usar o Auto Scaling na AWS (Linux) 

## Objetivo

- Criar uma instância do EC2 usando um comando da AWS CLI.

- Criar uma AMI usando a AWS CLI.

- Criar um modelo de execução do Amazon EC2.

- Criar uma configuração de execução do Amazon EC2 Auto Scaling.

- Configurar as políticas de scaling e criar um grupo do Auto Scaling para reduzir ou aumentar a quantidade de servidores com base em uma carga variável.

### Diagrama do fluxo 

Fluxo inicial:

<img width="598" alt="antes" src="https://github.com/user-attachments/assets/1e790460-6711-412f-a003-444b66d354cb" />


Fluxo Final

<img width="852" alt="depois" src="https://github.com/user-attachments/assets/1a331c6c-f945-433a-b633-408171652ab6" />

# Configuração de Auto Scaling no Amazon EC2

## Resumo
Este documento detalha o processo de configuração do Auto Scaling no Amazon EC2, incluindo a criação de uma AMI personalizada, a configuração de um Application Load Balancer e a implementação de um grupo de Auto Scaling para escalabilidade automática das instâncias.

## Tarefa 1: Criar uma AMI para o Amazon EC2 Auto Scaling

### 1.1 Conectar à instância Command Host
- Acesse o **Console de Gerenciamento da AWS** e abra o **EC2**.
- Selecione a instância **Command Host** e conecte-se via **EC2 Instance Connect**.

### 1.2 Configurar a AWS CLI
```sh
curl http://169.254.169.254/latest/dynamic/instance-identity/document | grep region
aws configure
```
- Insira `us-west-2` como região padrão e `json` como formato de saída.
- Navegue até o diretório necessário:
```sh
cd /home/ec2-user/
```

### 1.3 Criar uma instância do EC2
```sh
aws ec2 run-instances --key-name KEYNAME --instance-type t3.micro --image-id AMIID --user-data file:///home/ec2-user/UserData.txt --security-group-ids HTTPACCESS --subnet-id SUBNETID --associate-public-ip-address --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]' --output text --query 'Instances[*].InstanceId'
```
- Aguarde a inicialização:
```sh
aws ec2 wait instance-running --instance-ids NEW-INSTANCE-ID
```
- Obtenha o **DNS público**:
```sh
aws ec2 describe-instances --instance-id NEW-INSTANCE-ID --query 'Reservations[0].Instances[0].NetworkInterfaces[0].Association.PublicDnsName'
```

### 1.4 Criar uma AMI personalizada
```sh
aws ec2 create-image --name WebServerAMI --instance-id NEW-INSTANCE-ID
```

## Tarefa 2: Criar um ambiente de Auto Scaling

### 2.1 Criar um Application Load Balancer
- No **EC2**, selecione **Balanceadores de carga** > **Criar balanceador de carga**.
- Escolha **Application Load Balancer** e configure:
  - Nome: `WebServerELB`
  - Mapeamentos: **Sub-rede pública 1** e **Sub-rede pública 2**
  - Grupo de segurança: `HTTPAccess`
  - Criar grupo de destino: `webserver-app`

### 2.2 Criar um modelo de execução
- No **EC2**, selecione **Modelos de execução** > **Criar modelo de execução**.
- Configurações:
  - Nome: `web-app-launch-template`
  - AMI: `WebServerAMI`
  - Tipo de instância: `t3.micro`
  - Grupo de segurança: `HTTPAccess`

### 2.3 Criar um grupo do Auto Scaling
- No **EC2**, selecione **Grupos do Auto Scaling** > **Criar grupo do Auto Scaling**.
- Configurações:
  - Nome: `Web App Auto Scaling Group`
  - VPC: `VPC do laboratório`
  - Sub-redes: **Sub-rede privada 1** e **Sub-rede privada 2**
  - Grupo de destino: `webserver-app | HTTP`
  - Capacidade mínima: `2`, máxima: `4`, desejada: `2`
  - Política de escalabilidade: **Média de utilização da CPU = 50%**

## Tarefa 3: Verificar a configuração do Auto Scaling
- No **EC2**, verifique se duas instâncias `WebApp` foram criadas e estão íntegras.
- No **Load Balancer**, confirme a saúde das instâncias no grupo de destino `webserver-app`.

## Tarefa 4: Testar a configuração do Auto Scaling
- Acesse o **Nome do DNS do balanceador de carga** no navegador.
- Clique em **Iniciar stress**.
- No **EC2**, verifique a criação de novas instâncias no grupo do Auto Scaling.

## Conclusão
- Criação de instâncias EC2 e AMI via AWS CLI.
- Configuração de um Load Balancer e Auto Scaling Group.
- Teste de escalabilidade baseado no uso da CPU.
