# Criar uma EC2 

## Objetivo

- Executar um servidor web com proteção contra encerramento ativada

- Monitorar sua instância do EC2

- Modificar o grupo de segurança que seu servidor web está usando para permitir acesso HTTP

- Redimensionar sua instância do Amazon EC2 de acordo com a necessidade

- Explorar os limites do EC2

- Testar a proteção contra encerramento

- Encerrar a instância do EC2

### Diagrama do fluxo 

![EDN-EC2-CLI drawio](https://github.com/user-attachments/assets/d5f44b3f-e111-4773-a76c-2ab03deadd50)


## **Tarefa 1: Executar a Instância do Amazon EC2**
### **1. Acessar o EC2 no AWS Console**
- Navegue até **EC2** e clique em **Launch instance**.

### **2. Escolher uma AMI (Amazon Machine Image)**
- Selecione **Amazon Linux 2 AMI**.

### **3. Escolher um Tipo de Instância**
- Selecione **t3.micro** (2 vCPUs, 1 GiB RAM).

### **4. Configurar a Instância**
- Em **Network**, selecione **Lab VPC**.
- Ative **Protect against accidental termination**.
- Adicione um **User Data Script**:
  
  ```bash
  #!/bin/bash
  yum -y install httpd
  systemctl enable httpd
  systemctl start httpd
  echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
  ```

### **5. Adicionar Armazenamento**
- Volume padrão: **8 GiB**.

### **6. Adicionar Tags**
- Key: **Name** | Value: **Web Server**.

### **7. Configurar o Grupo de Segurança**
- Nome: **Web Server security group**.
- **Excluir a regra SSH** para reforçar a segurança.

### **8. Lançar a Instância**
- Escolha **Proceed without a key pair**.
- Inicie a instância e aguarde **running** e **2/2 status checks passed**.

---

## **Tarefa 2: Monitorar a Instância**
### **1. Verificar Status Checks**
- **System reachability** e **Instance reachability** devem estar aprovadas.

### **2. Monitoramento no CloudWatch**
- Acesse a aba **Monitoring** para visualizar métricas.

### **3. Obter Screenshot da Instância**
- Menu **Actions > Monitor and troubleshoot > Get Instance Screenshot**.

---

## **Tarefa 3: Atualizar o Grupo de Segurança e Acessar o Servidor Web**
### **1. Copiar o Endereço IPv4 Público** da instância.
### **2. Tentar acessar no navegador** – Deve falhar.
### **3. Atualizar o Grupo de Segurança**
- Tipo: **HTTP** | Fonte: **Anywhere**.
### **4. Atualizar o Navegador** e visualizar **Hello From Your Web Server!**.

---

## **Tarefa 4: Redimensionar a Instância – Tipo e Volume**
### **1. Parar a Instância**
- **Instance state > Stop instance**.

### **2. Alterar o Tipo da Instância**
- **Instance Settings > Change Instance Type**.
- Mudar para **t3.small** (mais memória).

### **3. Modificar o Volume EBS**
- **Volumes > Modify Volume**.
- Aumentar de **8 GiB para 10 GiB**.

### **4. Iniciar a Instância**
- **Instance state > Start instance**.

---

## **Tarefa 5: Explorar os Limites do EC2**
- Navegue até **Limits** no painel esquerdo.
- Observe **limites de instâncias por região** e como solicitar aumento.

---

## **Tarefa 6: Testar a Proteção Contra Encerramento**
### **1. Tentar Encerrar a Instância**
- O sistema impede devido à **Termination Protection**.

### **2. Desativar Proteção**
- **Actions > Instance Settings > Change termination protection**.
- Desmarque **Enable** e salve.

### **3. Encerrar a Instância**
- **Actions > Instance State > Terminate**.

---


