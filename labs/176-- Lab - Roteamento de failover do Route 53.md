# Roteamento de failover do Amazon Route 53 


## Objetivo

- Configurar uma health check do Route 53 que envie e-mails quando a integridade de um endpoint HTTP deixar de ser íntegro.

- Configurar o roteamento de failover no Route 53..


### Diagrama do fluxo ✅

![failover-arch](https://github.com/user-attachments/assets/be4e0d93-bef6-42d9-b5e1-384a1a2697d1)


# Configuração de Failover e Health Check no AWS Route 53

## Tarefa 1: Confirmar os sites da cafeteria

1. **Copiar Parâmetros**:
   - Acesse o painel do AWS CloudFormation.
   - Selecione "Detalhes" e, em seguida, "Mostrar" em AWS. Isso abrirá um painel de Credenciais.
   - Copie os seguintes parâmetros e cole-os em um editor de texto para uso posterior:
     - **CafeInstance1IPAddress**: Endereço IP público da instância 1.
     - **PrimaryWebSiteURL**: URL do site principal da cafeteria.
     - **SecondaryWebsiteURL**: URL do site secundário da cafeteria.
     - **CafeInstance2IPAddress**: Endereço IP público da instância 2.

2. **Acessar o Console de EC2**:
   - No Console de Gerenciamento da AWS, vá até "Serviços" e selecione **EC2**.
   - No painel de navegação à esquerda, clique em **Instâncias**.
   - Confirme que duas instâncias foram criadas:
     - **CafeInstance1**: em execução na sub-rede pública 1 (us-west-2a).
     - **CafeInstance2**: em execução na sub-rede pública 2 (us-west-2b).

3. **Verificar os Sites**:
   - Abra uma nova guia do navegador e cole o **PrimaryWebSiteURL**. O site principal da cafeteria deve carregar, mostrando a aplicação em execução na instância **CafeInstance1**.
   - Abra outra guia e cole o **SecondaryWebsiteURL**. O site secundário deve carregar, confirmando que a aplicação está em execução na **CafeInstance2**.

4. **Verificar Funcionalidade**:
   - Em qualquer um dos sites, selecione **Menu**, escolha um item e clique em **Enviar pedido**.
   - Na página de **Confirmação do pedido**, verifique se o horário do pedido corresponde ao fuso horário da instância do servidor. Isso confirma que a aplicação está funcionando nas duas instâncias com as configurações corretas.

---

## Tarefa 2: Configurar uma Health Check do Route 53

1. **Acessar o Console do Route 53**:
   - No Console da AWS, no menu **Serviços**, selecione **Route 53**.
   - No painel de navegação à esquerda, clique em **Verificações de integridade**.

2. **Criar uma Health Check**:
   - Clique em **Criar verificação de integridade**.
   - Configure as opções com os seguintes valores:
     - **Nome**: `Primary-Website-Health`.
     - **O que monitorar**: Selecione **Endpoint**.
     - **Especifique o endpoint por**: Selecione **Endereço IP**.
     - **Endereço IP**: Cole o valor de **CafeInstance1IPAddress**.
     - **Caminho**: Digite `cafe`.
   - Expanda a seção **Configuração avançada** e configure as opções como segue:
     - **Intervalo de solicitações**: Selecione **Rápido (10 segundos)**.
     - **Limite de falha**: Digite `2`.

3. **Configurar Notificações**:
   - Selecione **Próximo**.
   - **Criar alarme**: Selecione **Sim**.
   - **Enviar notificação para**: Selecione **Novo tópico do SNS**.
   - **Nome do tópico**: `Primary-Website-Health`.
   - **Endereço de e-mail do destinatário**: Insira um e-mail válido para receber notificações de falhas.
   - Clique em **Criar verificação de integridade**.

4. **Monitorar Status da Health Check**:
   - A health check começará a monitorar o site, solicitando periodicamente o nome de domínio configurado.
   - Pode levar até um minuto para que o status seja atualizado para "Íntegro". Use o ícone de atualização para atualizar o status.

5. **Confirmar Assinatura de Notificação**:
   - Verifique o e-mail e clique no link **Confirmar assinatura** para concluir a configuração da notificação por e-mail.
   
---

## Tarefa 3: Configurar Registros do Route 53

### Tarefa 3.1: Criar um Registro A para o Site Principal

1. **Acessar Zonas Hospedadas**:
   - No Console do Route 53, no painel de navegação à esquerda, selecione **Zonas hospedadas**.
   - Localize e selecione o nome de domínio **XXXXXX_XXXXXXXXXX.vocareum.training** (um domínio exclusivo gerado para sua conta da AWS).

2. **Criar um Registro A para o Site Principal**:
   - Clique em **Criar registro** e configure as seguintes opções:
     - **Nome do registro**: `www`.
     - **Tipo de registro**: Selecione **A: Rotear tráfego para um endereço IPv4**.
     - **Valor**: Cole o valor de **CafeInstance1IPAddress**.
     - **TTL (segundos)**: Digite `15`.
     - **Política de roteamento**: Selecione **Failover**.
     - **Tipo de registro de failover**: Selecione **Primário**.
     - **ID da verificação de integridade**: Selecione a health check **Integridade do site principal**.
     - **ID do registro**: Insira `FailoverPrimary`.

3. **Criar o Registro**:
   - Clique em **Criar registros**. O novo registro será exibido na lista de Zonas hospedadas.

### Tarefa 3.2: Criar um Registro A para o Site Secundário

1. **Criar um Registro A para o Site Secundário**:
   - Clique em **Criar registro** e configure as seguintes opções:
     - **Nome do registro**: `www`.
     - **Tipo de registro**: Selecione **A: Rotear tráfego para um endereço IPv4**.
     - **Valor**: Cole o valor de **CafeInstance2IPAddress**.
     - **TTL (segundos)**: Digite `15`.
     - **Política de roteamento**: Selecione **Failover**.
     - **Tipo de registro de failover**: Selecione **Secundário**.
     - **ID do registro**: Insira `FailoverSecondary`.

2. **Criar o Registro**:
   - Clique em **Criar registros**. O novo registro será exibido na lista de Zonas hospedadas.

---

## Tarefa 4: Verificar a Resolução de DNS

1. **Acessar o Registro A**:
   - No painel de Zonas hospedadas, marque o registro A recém-criado para o site principal.
   - No painel de detalhes do registro, copie o **Nome do registro**.

2. **Verificar no Navegador**:
   - Abra uma nova guia no navegador, cole o **Nome do registro** seguido de `/cafe` no final do URL. Exemplo:
     ```
     http://www.XXXXXX_XXXXXXXXXX.vocareum.training/cafe/
     ```
   - A página do site principal da cafeteria deve carregar, mostrando a **Zona de Disponibilidade**.

---

## Tarefa 5: Verificar a Funcionalidade do Failover

1. **Simular Falha no Site Principal**:
   - No Console da AWS, vá até **EC2** e selecione **Instâncias**.
   - Selecione a instância **CafeInstance1** e, no menu **Estado da instância**, escolha **Interromper instância**.
   - Confirme a interrupção da instância **CafeInstance1**.

2. **Monitorar Status da Health Check**:
   - A health check do Route 53 detectará que **CafeInstance1** está fora de serviço e, em seguida, o tráfego será redirecionado para **CafeInstance2**.
   - No Console do Route 53, acesse **Verificações de integridade** e selecione **Integridade do site principal**.
   - Verifique o status. O status será atualizado para **Não íntegro** após a falha na instância principal.

3. **Verificar Failover no Navegador**:
   - Abra a página de **vocareum_XXXXXX_XXXXXXXXXX.training/cafe** novamente e atualize a página.
   - O site agora deve mostrar informações sobre a **Zona de Disponibilidade** de **CafeInstance2** (por exemplo, `us-west-2b`).

4. **Receber Notificação de Falha**:
   - Verifique seu e-mail para uma notificação da AWS sobre o alarme de falha. O e-mail será intitulado **“ALARM: Primary-Website-Health-awsroute53-...”**.

---

## Conclusão

- Configuração da health check no Route 53, com alertas por e-mail quando um endpoint HTTP deixa de ser íntegro.
- Implementação do roteamento de failover no Route 53, garantindo alta disponibilidade entre duas zonas de disponibilidade.
