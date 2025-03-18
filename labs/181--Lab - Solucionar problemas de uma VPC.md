# Solução de problemas na VPC

## Objetivo 

- Criar logs de fluxo da VPC.

- Solucionar problemas de configuração da VPC.

- Analisar logs de fluxo.

### Diagrama do fluxo


<img width="729" alt="Architecture" src="https://github.com/user-attachments/assets/9b5d46a9-a925-4ecc-86f3-5353aba43e34" />


# Tarefa 1: Conectar-se à instância CLI Host

Nesta tarefa, você usará o EC2 Instance Connect para se conectar à instância CLI Host. Você usará essa instância para executar comandos da AWS Command Line Interface (AWS CLI).

No Console de Gerenciamento da AWS, na barra Pesquisar, insira e escolha EC2 para abrir o Console de Gerenciamento do EC2.

No painel de navegação, selecione Instâncias.

Na lista de instâncias, selecione a instância CLI Host.

Selecione Conectar-se.

Na guia EC2 Instance Connect, selecione Conectar-se.

Essa opção abre uma nova guia do navegador que mostra a janela do terminal do EC2 Instance Connect.

**Observação:** se preferir usar um cliente SSH para se conectar à instância do EC2, consulte as orientações em Conecte-se à sua instância do Linux.

Use essa janela do terminal para concluir as tarefas do laboratório. Se o terminal deixar de responder, atualize o navegador ou use as etapas desta tarefa para se conectar novamente.

Agora que você já estabeleceu conexão com a instância CLI Host, é possível configurar e usar a AWS CLI para chamar os serviços da AWS.

## Tarefa 1.1: Configurar a AWS CLI na instância CLI Host

Para configurar o perfil da AWS CLI com credenciais, no terminal do EC2 Instance Connect, execute o seguinte comando:

```bash
aws configure
```

Nos prompts, copie os valores a seguir que você colou no editor de texto e cole-os na janela do terminal conforme as instruções.

- **AWS Access Key ID (ID da chave de acesso da AWS):** insira o valor de AccessKey.
- **AWS Secret Access Key (Chave de acesso secreta da AWS):** insira o valor de SecretKey.
- **Default region name (Nome da região padrão):** insira us-west-2.
- **Default output format (Formato de saída padrão):** insira json.

Execute os comandos da CLI nessa janela do terminal de CLI Host conforme as instruções ao longo do laboratório.

# Tarefa 2: Criar logs de fluxo da VPC

Nesta tarefa, você criará um bucket do S3 para publicar os dados de logs de fluxo da VPC. Depois, você criará logs de fluxo da VPC em VPC1 para capturar informações sobre o tráfego IP entre as interfaces de rede em VPC1. Os logs de fluxo serão publicados no bucket do S3.

Para criar o bucket do S3 onde os logs de fluxo serão publicados, execute o comando a seguir. No comando, substitua ###### por seis números aleatórios:

```bash
aws s3api create-bucket --bucket flowlog###### --region 'us-west-2' --create-bucket-configuration LocationConstraint='us-west-2'
```

A saída em formato JSON, semelhante à seguinte, mostra o local de um bucket: `http://flowlog######.s3.amazonaws.com`

Neste comando, `flowlog######` é o nome do seu bucket. Você usará esse nome de bucket em uma etapa posterior.

**Observação:** se você receber uma mensagem de erro indicando que o nome do bucket já existe, use um conjunto diferente de números aleatórios para substituir ###### e execute o comando novamente.

Para obter o ID de VPC1 e criar logs de fluxo da VPC, execute o seguinte comando:

```bash
aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,Tags[?Key==`Name`].Value,CidrBlock]' --filters "Name=tag:Name,Values='VPC1'"
```

A saída em formato JSON, semelhante à seguinte, mostra o ID da VPC: `vpc-01edacbe1c31959d2`.

Para criar logs de fluxo da VPC em VPC1, execute o comando a seguir. No comando, substitua `<flowlog.######>` pelo nome do bucket das etapas anteriores e substitua `<vpc-id>` pelo ID de VPC1 da etapa anterior.

```bash
aws ec2 create-flow-logs --resource-type VPC --resource-ids <vpc-id> --traffic-type ALL --log-destination-type s3 --log-destination arn:aws:s3:::<flowlog######>
```

A saída do comando retorna `FlowLogIds` e um `ClientToken`.

**Observação:** se for exibida uma mensagem de falha, ignore-a.

Para confirmar se o log de fluxo foi criado, execute o seguinte comando:

```bash
aws ec2 describe-flow-logs
```

A saída do comando deve mostrar que um único log de fluxo foi criado com um `FlowLogStatus ACTIVE` (ATIVO) e um destino de log que aponta para o bucket do S3.

Agora que o log de fluxo foi criado, você pode prosseguir para a próxima tarefa, que envolve um pouco de solução de problemas.

# Tarefa 3: Solucionar problemas de configuração da VPC para permitir acesso aos recursos.

Nesta tarefa, você analisará o acesso à instância do servidor web e solucionará alguns problemas de rede. Lembre-se de que a instância Café Web Server é executada na sub-rede pública de VPC1. Consulte o diagrama no início deste laboratório para ver detalhes sobre como a rede deve ser configurada.

Em seu editor de texto, copie o endereço IP de WebServerIP e cole-o em uma nova guia do navegador. 

Após alguns instantes, a página não é carregada e você recebe uma mensagem indicando que o site não pode ser acessado ou que a conexão expirou. Essa mensagem é esperada.

Deixe essa guia do navegador aberta para que você possa retornar a ela mais tarde.

No terminal de CLI Host, execute o comando a seguir para encontrar detalhes sobre a instância do servidor web. No comando, substitua `<WebServerIP>` pelo endereço de WebServerIP utilizado nas etapas anteriores:

```bash
aws ec2 describe-instances --filter "Name=ip-address,Values='<WebServerIP>'"
```

Um documento JSON grande é retornado, fornecendo mais detalhes do que você precisa para a solução dos problemas.

Para retornar apenas detalhes relevantes, filtre os resultados do lado do cliente (usando o parâmetro query). O comando na próxima etapa retorna apenas o estado da instância, o endereço IP privado, o ID da instância, os grupos de segurança aplicados a ela, a sub-rede na qual ela é executada e o nome do par de chaves associado a ela. 

Para filtrar os resultados, execute o comando a seguir. No comando, substitua `<WebServerIP>` pelo mesmo endereço de WebServerIP utilizado nas etapas anteriores: 

```bash
aws ec2 describe-instances --filter "Name=ip-address,Values='<WebServerIP>'" --query 'Reservations[*].Instances[*].[State,PrivateIpAddress,InstanceId,SecurityGroups,SubnetId,KeyName]'
```

Os resultados do comando indicam que a instância está em execução e retornam informações adicionais que poderão ser usadas posteriormente.

Em seguida, tente estabelecer uma conexão SSH com a instância do servidor web usando o EC2 Instance Connect.

Na guia do navegador com o Console de Gerenciamento da AWS, na barra Pesquisar, insira e escolha EC2 para abrir o Console de Gerenciamento do EC2.

No painel de navegação, selecione Instâncias.

Na lista de instâncias, selecione a instância Café Web Server.

Selecione Conectar-se.

Na guia EC2 Instance Connect, selecione Conectar-se.

Após alguns segundos, a tentativa de conexão falha. Você recebe um erro na janela do navegador que diz: “Falha ao conectar-se à sua instância.” 

Esse comportamento está dentro do esperado.

## Desafio n.º 1 da solução de problemas

Você estabeleceu que a instância do servidor web está em execução, mas a página da web não está carregando. Qual poderia ser o problema?

Desafie-se a conduzir sua investigação usando apenas o acesso programático da AWS CLI. Evite usar o Console de Gerenciamento da AWS.

**Dicas:**
- Use o utilitário nmap para verificar quais portas estão abertas na instância do EC2 do servidor web. 
- Para fazer isso, comece instalando o utilitário na instância CLI Host executando o comando:

```bash
sudo yum install -y nmap
```

- Depois, execute o comando:

```bash
nmap <WebServerIP>
```

Se o nmap não puder encontrar portas abertas, será que pode haver algo mais bloqueando o acesso à instância?

- Verifique os detalhes do grupo de segurança usando o comando:

```bash
aws ec2 describe-security-groups
```

Talvez seja útil analisar os resultados do comando se usar o parâmetro `group-ids`. Esse valor também está disponível no editor de texto (WebServerSgId) com os outros valores que você usou neste laboratório. 

Você pode usar o seguinte comando para consultar a conectividade com a porta 22:

```bash
aws ec2 describe-security-groups --group-ids 'WebServerSgId' 
```

- Você também pode usar o comando `describe-instances` para retornar o ID do grupo de segurança.

Depois de executar o comando `describe-security-groups`, analise a saída resultante. 

As configurações do grupo de segurança aplicadas à instância do EC2 do servidor web parecem estar permitindo conectividade com a porta 22? 

Verifique as configurações da tabela de rota associada à sub-rede em que o servidor web está em execução.

Use o comando:

```bash
aws ec2 describe-route-tables
```

Ao executar esse comando, talvez seja útil aplicar um filtro como este exemplo:

```bash
--filter "Name=association.subnet-id,Values='<VPC1PubSubnetID>'"
```

Nesse comando, substitua `<VPC1PubSubnetID>` pelo valor real do ID da sub-rede que está no editor de texto.

Você pode utilizar o seguinte comando:

```bash
aws ec2 describe-route-tables  --route-table-ids 'VPC1PubRouteTableId' --filter "Name=association.subnet-id,Values='VPC1PubSubnetID'" 
```

Ao analisar a saída do comando `describe-route-tables`, lembre-se de que a sub-rede está identificada como pública.

Você percebe algum problema com as rotas? 

Se você tiver que definir uma nova rota, use o comando:

```bash
aws ec2 create-route
```

Você deve conhecer o `route-table-id` e o `gateway-id` para criar uma rota com êxito. Esses dois valores estão disponíveis no editor de texto. Você também deve ter o `route-table-id` de quando executou o comando `describe-route-tables` anteriormente.

Use o seguinte comando para criar rotas conforme necessário:

 No comando a seguir, substitua VPC1PubRouteTableId e VPC1GatewayId pelos valores que estão no editor de texto e execute o comando ajustado. 

```bash
aws ec2 create-route --route-table-id 'VPC1PubRouteTableId' --gateway-id  'VPC1GatewayId' --destination-cidr-block '0.0.0.0/0' 
```
- Você também pode usar o comando aws ec2 describe-internet-gateways para obter o gateway-id. Você também poderá precisar especificar outros parâmetros para executar o comando com sucesso.

Depois de achar que resolveu o problema, retorne à guia do navegador onde você tentou carregar a página do servidor web e atualize a página da web. A página do navegador deve exibir uma mensagem que diz: “Hello From Your Web Server!”.

## Desafio n.º 2 da solução de problemas

Para resolver o problema de conexão SSH à instância do servidor web:

1. **Verifique as configurações da lista de controle de acesso de rede (ACL de rede)**:
   ```bash
   aws ec2 describe-network-acls --filter "Name=association.subnet-id,Values='<VPC1PublicSubnetID>'" --query 'NetworkAcls[*].[NetworkAclId,Entries]'
   ```

2. **Excluir entradas problemáticas da ACL de rede**:
   ```bash
   aws ec2 delete-network-acl-entry --network-acl-id <network-acl-id> --rule-number <rule-number>
   ```
   Substitua `<network-acl-id>` e `<rule-number>` pelos valores recuperados anteriormente.

3. **Tente se conectar novamente à instância do servidor web usando o EC2 Instance Connect**:
   - **Verifique a conexão SSH**:
     ```bash
     ssh ec2-user@<WebServerIP>
     ```
     Após conectar-se, execute o comando `hostname` para confirmar que você se conectou à instância correta.

## Tarefa 4: Analisar logs de fluxo

#### Tarefa 4.1: Baixar e extrair os logs de fluxo

1. **Crie um diretório local no terminal de CLI Host**:
   ```bash
   mkdir flowlogs
   ```
   
2. **Navegue para o novo diretório**:
   ```bash
   cd flowlogs
   ```
   
3. **Liste os buckets do S3 para encontrar o nome do bucket**:
   ```bash
   aws s3 ls
   ```
   
4. **Baixe os logs de fluxo do bucket do S3**:
   ```bash
   aws s3 cp s3://<flowlog######>/ . --recursive
   ```
   Substitua `<flowlog######>` pelo nome do bucket.

5. **Navegue para o subdiretório onde os logs foram baixados**:
   ```bash
   cd <AWSLogs/AccountID/vpcflowlogs/us-west-2/yyyy/mm/dd/>
   ```
   
6. **Extraia os arquivos de log**:
   ```bash
   gunzip *.gz
   ```

#### Tarefa 4.2: Analisar os logs

1. **Examine a estrutura dos logs**:
   ```bash
   head <file name>
   ```
   Substitua `<file name>` pelo nome de um arquivo retornado pelo comando `ls`.

2. **Procure eventos REJECT nos logs**:
   ```bash
   grep -rn REJECT .
   ```
   
3. **Conte o número de eventos REJECT**:
   ```bash
   grep -rn REJECT . | wc -l
   ```
   
4. **Refine a pesquisa para a porta 22 (SSH)**:
   ```bash
   grep -rn 22 . | grep REJECT
   ```

5. **Determine o endereço IP da sua máquina local**:
   - No Console de Gerenciamento da AWS, acesse os **Grupos de segurança**.
   - Edite as **Regras de entrada** do grupo **WebSecurityGroup** e adicione uma regra com a origem **Meu IP** para capturar o endereço IP.
   - Copie o endereço IP preenchido automaticamente, sem o sufixo `/32`.

6. **Refine a pesquisa nos logs para o seu endereço IP**:
   ```bash
   grep -rn 22 . | grep REJECT | grep <ip-address>
   ```
   Substitua `<ip-address>` pelo endereço IP capturado.

7. **Confirme o ID da interface de rede**:
   ```bash
   aws ec2 describe-network-interfaces --filters "Name=association.public-ip,Values='<WebServerIP>'" --query 'NetworkInterfaces[*].[NetworkInterfaceId,Association.PublicIp]'
   ```
   Substitua `<WebServerIP>` pelo endereço IP da instância do servidor web.

8. **Converta carimbos de data/hora Unix em formato legível**:
   ```bash
   date -d @<timestamp>
   ```


   


