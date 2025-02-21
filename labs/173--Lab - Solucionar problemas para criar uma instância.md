#   Solucionar problemas com a criação de uma instância do EC2

## Objetivos 
- Iniciar uma instância do EC2 usando a AWS CLI.

- Solucionar problemas em comandos da AWS CLI e nas configurações do serviço Amazon EC2 usando dicas básicas de solução de problemas e o utilitário nmap de código aberto.


## Diagrama do fluxo 

<img width="1568" alt="Architecture" src="https://github.com/user-attachments/assets/5148461b-9792-4fc9-9ea5-381ba381602e" />

# Guia de Configuração e Solução de Problemas no AWS EC2

## Tarefa 1: Conectar-se à Instância CLI Host  
Nesta etapa, utilizaremos o **EC2 Instance Connect** para estabelecer uma conexão com a instância **CLI Host**, que foi criada automaticamente no provisionamento do laboratório. Essa instância será usada para executar comandos da **AWS CLI**.

### Passos:
1. Acesse o **Console de Gerenciamento da AWS**.
2. Na barra de pesquisa, digite **EC2** e selecione a opção para abrir o **Console de Gerenciamento do Amazon EC2**.
3. No painel de navegação à esquerda, clique em **Instâncias**.
4. Na lista de instâncias, localize e selecione a instância chamada **CLI Host**.
5. Acima da lista de instâncias, clique em **Conectar-se**.
6. Na guia **EC2 Instance Connect**, clique em **Conectar-se** para abrir o terminal.

> **Observação:** Se preferir usar um cliente SSH para se conectar à instância EC2, consulte a documentação da AWS sobre **Como conectar-se a uma instância do Linux**.

Agora que a conexão foi estabelecida, podemos configurar e utilizar a **AWS CLI** para interagir com os serviços da AWS.

---

## Tarefa 2: Configurar a AWS CLI  
A **AWS CLI** já vem pré-instalada nas AMIs do **Amazon Linux**. Precisamos apenas configurar as credenciais de acesso para operar os serviços AWS.

### Passos:
1. Execute o seguinte comando para iniciar a configuração:
   ```sh
   aws configure
   ```
2. Insira as informações solicitadas:
   - **AWS Access Key ID**: Obtida na seção **Detalhes** do laboratório.
   - **AWS Secret Access Key**: Obtida na mesma seção.
   - **Default region name**: Use o valor **LabRegion** fornecido.
   - **Default output format**: Insira `json`.

Agora, a **AWS CLI** está configurada e pronta para ser utilizada.

---

## Tarefa 3: Criar uma Instância EC2 via AWS CLI  
Nesta etapa, analisaremos e corrigiremos um **script de shell** para provisionar uma instância **LAMP (Linux, Apache, MySQL, PHP)** via AWS CLI.

### 3.1 Analisar o Script de Criação  
Antes de modificar o script, é uma boa prática criar um **backup**.

#### Passos:
1. Acesse o diretório onde o script está localizado:
   ```sh
   cd ~/sysops-activity-files/starters  
   ```
2. Crie um backup do script original:
   ```sh
   cp create-lamp-instance-v2.sh create-lamp-instance.backup  
   ```
3. Abra o script para visualização:
   ```sh
   view create-lamp-instance-v2.sh
   ```
4. Estrutura do script:
   - **Linhas 1-6:** Declaração do interpretador Bash (`#!/bin/bash`).
   - **Linhas 7-11:** Define o tamanho da instância como `t3.small`.
   - **Linhas 16-29:** Identifica a **VPC da cafeteria** e armazena o **ID da VPC**.
   - **Linhas 31-55:** Obtém valores de **sub-rede, par de chaves e ID da AMI**.
   - **Linhas 57-122:** Verifica se já existem instâncias ou grupos de segurança com nomes similares e os exclui, se necessário.
   - **Linhas 124-152:** Criação do **grupo de segurança** com regras para as portas **22 (SSH)** e **80 (HTTP)**.
   - **Linhas 154-168:** Criação da instância EC2 e captura dos detalhes da instância.
   - **Linhas 179-188:** Aguarda a atribuição de um **endereço IP público** para a instância.

5. Para visualizar o **script de dados do usuário**, execute:
   ```sh
   cat create-lamp-instance-userdata-v2.txt
   ```
   - Esse script instala o **Apache, PHP e MariaDB** na instância após a inicialização.

### 3.2 Executar o Script  
Execute o script para iniciar a criação da instância:
```sh
./create-lamp-instance-v2.sh
```
> O script contém erros intencionais. Precisamos identificá-los e corrigi-los.

### 3.3 Solução de Problemas  

#### Problema #1: AMI não encontrada  
**Erro:** `InvalidAMIID.NotFound`  
- Verifique a linha onde o ID da **AMI** é definido no script.
- Confirme se a **região** utilizada na criação da instância está correta.
- Corrija o erro no script e tente executá-lo novamente.

#### Problema #2: Site não acessível  
Após a criação da instância, o site não carrega no navegador.

1. Verifique se a **porta 80** está aberta no grupo de segurança.
2. Confirme se o servidor web está rodando:
   ```sh
   sudo systemctl status httpd
   ```
3. Instale e use o `nmap` para verificar portas abertas:
   ```sh
   sudo yum install -y nmap  
   nmap -Pn <public-ip>
   ```
4. Caso necessário, ajuste as regras de segurança e reinicie o servidor web.
5. Para verificar logs do **cloud-init** e validar a instalação:
   ```sh
   sudo tail -f /var/log/cloud-init-output.log
   ```

---

## Tarefa 4: Verificar a Funcionalidade do Site  
1. Acesse no navegador:
   ```
   http://<public-ip>/cafe
   ```
2. **Testar funcionalidades**:
   - Acesse o **menu** e faça pedidos.
   - Verifique o **histórico de pedidos**.
   - Confirme que os pedidos estão sendo armazenados no banco de dados.

---

## Conclusão  
- Criamos e configuramos uma instância EC2 via AWS CLI.  
- Identificamos e corrigimos erros no script de criação.  
- Verificamos portas, logs e regras de segurança para solução de problemas.  
- Validamos o funcionamento do site implantado.  

🎉 Parabéns pela conclusão do laboratório!


---
