# Configurar uma VPC


## Objetivo

- Criar uma VPC com uma sub-rede privada e uma sub-rede pública, um gateway de internet e um gateway NAT.

- Configurar as tabelas de rota associadas às sub-redes para o tráfego local e para o tráfego vinculado à internet usando um gateway de internet e um gateway NAT.

- Iniciar um servidor bastion na sub-rede pública.

- Usar um servidor bastion para fazer login em uma instância em uma sub-rede privada.

### Diagrama do fluxo

![architecture](https://github.com/user-attachments/assets/27c43ba0-404c-4280-82e6-44ec8bc17b69)

# Configuração de VPC na AWS

## Tarefa 1: Criar uma VPC
1. **Acesse o Console de Gerenciamento da AWS** e pesquise por **VPC** na barra de pesquisa.
2. No painel de navegação à esquerda, em **Nuvem privada virtual**, selecione **Suas VPCs**.
3. Uma VPC padrão será exibida. Clique em **Criar VPC** e configure as seguintes opções:
   - **Recursos a serem criados**: Selecione **Somente VPC**.
   - **Tag de nome**: Insira **Lab VPC**.
   - **IPv4 CIDR block**: Insira **10.0.0.0/16**.
   - **IPv6 CIDR block**: Selecione **Nenhum**.
   - **Locação**: Escolha **Padrão**.
   - **Tags**: Mantenha as tags sugeridas.
4. Após a criação da VPC, você verá uma mensagem confirmando a criação da VPC, como: "Você criou com sucesso o vpc-NNNNNNNNNNNNNNNN / Lab VPC".
5. Clique em **Ações** e escolha **Editar configurações da VPC**.
6. Na seção **Configurações de DNS**, selecione **Habilitar nomes de host DNS**.
7. Clique em **Salvar**.

---

## Tarefa 2: Criar Sub-redes

### 2.1: Criar uma Sub-rede Pública
1. No painel de navegação à esquerda, em **Nuvem privada virtual**, selecione **Sub-redes**.
2. Clique em **Criar sub-rede** e configure as seguintes opções:
   - **ID da VPC**: Escolha **Lab VPC**.
   - **Nome da sub-rede**: Insira **Public Subnet**.
   - **Zona de disponibilidade**: Escolha a primeira Zona de Disponibilidade na lista.
   - **IPv4 CIDR block**: Insira **10.0.0.0/24**.
3. Após a criação, configure a sub-rede pública para atribuir automaticamente um endereço IP público às instâncias EC2:
   - Selecione a sub-rede pública.
   - Clique em **Ações** e selecione **Editar configurações da sub-rede**.
   - Na seção **Configurações de atribuição automática de IP**, selecione **Habilitar a atribuição de endereço IPv4 público automaticamente**.
   - Clique em **Salvar**.

### 2.2: Criar uma Sub-rede Privada
1. Crie a sub-rede privada com as seguintes configurações:
   - **ID da VPC**: Escolha **Lab VPC**.
   - **Nome da sub-rede**: Insira **Private Subnet**.
   - **Zona de disponibilidade**: Escolha a primeira Zona de Disponibilidade na lista.
   - **IPv4 CIDR block**: Insira **10.0.2.0/23**.

---

## Tarefa 3: Criar um Gateway de Internet
1. No painel de navegação à esquerda, em **Nuvem privada virtual**, selecione **Gateways da Internet**.
2. Clique em **Criar Gateway da Internet** e insira **Lab IGW** como tag de nome.
3. Clique em **Criar Gateway da Internet**.
4. Selecione **Ações** e clique em **Associar à VPC**.
   - Sua sub-rede pública agora estará conectada à internet.

---

## Tarefa 4: Configurar Tabelas de Rota
1. No painel de navegação à esquerda, em **Nuvem privada virtual**, selecione **Tabelas de Rotas**.
2. Escolha a tabela de rotas associada à **Lab VPC** e edite o nome para **Private Route Table**.
3. Crie uma tabela de rota pública para direcionar o tráfego vinculado à internet para o **Gateway de Internet**:
   - **Nome**: **Public Route Table**.
   - **VPC**: Escolha **Lab VPC**.
4. Após criar a tabela de rota, adicione uma rota para direcionar o tráfego da internet (**0.0.0.0/0**) para o **Gateway de Internet**.
5. Associe essa tabela de rota à **sub-rede pública**:
   - Selecione **Associações de sub-rede**.
   - Clique em **Editar associações de sub-rede**.
   - Escolha a **Sub-rede pública** e clique em **Salvar associações**.

---

## Tarefa 5: Iniciar um Servidor Bastion na Sub-rede Pública
1. Acesse o **Console de Gerenciamento do EC2**.
2. Clique em **Executar instâncias** e configure as seguintes opções:
   - **Nome**: **Bastion Server**.
   - **Imagem de Máquina Amazon** (AMI): Escolha **Amazon Linux 2023**.
   - **Tipo de Instância**: **t3.micro**.
   - **Atribuição de IP público**: Selecione **Habilitar**.
   - **Grupo de segurança**: Crie um grupo de segurança chamado **Bastion Security Group**, permitindo acesso SSH de qualquer lugar.
3. Após a criação, a instância será chamada **Bastion Server** e estará na **sub-rede pública**.

---

## Tarefa 6: Criar um Gateway NAT
1. Acesse **Gateways NAT** no Console de Gerenciamento da AWS.
2. Crie um gateway NAT com as seguintes configurações:
   - **Nome**: **Lab NAT gateway**.
   - **Sub-rede**: Escolha **Sub-rede pública**.
   - **Alocar IP elástico**: Selecione esta opção.
3. Após a criação, configure a **sub-rede privada** para direcionar o tráfego para o **Gateway NAT**:
   - No painel de navegação, selecione **Tabelas de Rotas** e edite a **Tabela de Rota Privada**.
   - Adicione uma rota para **0.0.0.0/0** direcionando o tráfego para o **Gateway NAT**.
   - Clique em **Salvar alterações**.

---

## Desafio Opcional: Testar a Sub-rede Privada

### Iniciar uma Instância na Sub-rede Privada
1. Inicie uma nova instância na **sub-rede privada**:
   - **Nome**: **Private Instance**.
   - **AMI**: **Amazon Linux 2023**.
   - **Tipo de instância**: **t3.micro**.
   - **Configurações de rede**: Escolha **Lab VPC** e **Private Subnet**.
   - **Grupo de segurança**: Crie um grupo de segurança **Private Instance SG**, permitindo SSH de **10.0.0.0/16**.

### Conectar-se à Instância via Servidor Bastion
1. Acesse a instância **Bastion Server** e depois conecte-se à **instância privada** usando o comando `ssh`.
2. Teste a conectividade com a internet executando o comando `ping -c 3 amazon.com`. Você deve ver respostas de **amazon.com**.

---

## Conclusão

1. Criou uma VPC com sub-redes pública e privada, gateway de internet e gateway NAT.
2. Configurou tabelas de rota para tráfego local e internet.
3. Iniciou um servidor bastion na sub-rede pública.
4. Usou o servidor bastion para acessar recursos na sub-rede privada e testou a conectividade com a internet.
