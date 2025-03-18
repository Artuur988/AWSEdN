# Dimensionar e balancear a carga da arquitetura 🖥️

## Objetivo

Balanceador de carga
- Criar uma AMI de uma instância do EC2.

- Criar um balanceador de carga.

- Criar um modelo de execução e um grupo do Auto Scaling.

- Configurar um grupo do Auto Scaling para dimensionar novas instâncias em sub-redes privadas.

- Criar alarmes do Amazon CloudWatch para monitorar o desempenho da infraestrutura.

### Diagrama do fluxo 

<img width="1267" alt="FinalArchitecture" src="https://github.com/user-attachments/assets/9ab7c680-fb9c-4869-acf9-be4bab6b6855" />

# Auto Scaling na AWS

## Tarefa 1: Criar uma AMI para o Auto Scaling
1. Acesse o Console de Gerenciamento da AWS e selecione **EC2**.
2. Vá até **Instâncias** e selecione a instância **Web Server 1** (em execução).
3. No menu **Ações**, escolha **Imagem e modelos > Criar imagem**.
4. Configure:
   - Nome da imagem: `Web Server AMI`
   - Descrição (opcional): `Lab AMI for Web Server`
5. Selecione **Criar imagem** e anote o ID da AMI gerada.

---

## Tarefa 2: Criar um Balanceador de Carga
1. No Console EC2, vá até **Balanceadores de carga** e selecione **Criar balanceador de carga**.
2. Escolha **Application Load Balancer** e configure:
   - Nome: `LabELB`
   - VPC: `VPC do laboratório`
   - Sub-redes: `Sub-rede pública 1` e `Sub-rede pública 2`
   - Grupo de segurança: `Grupo de segurança da web`
3. Crie um **Grupo de destino**:
   - Nome: `lab-target-group`
   - Tipo de destino: `Instâncias`
4. Associe o grupo de destino ao balanceador de carga e finalize a criação.
5. Copie o **Nome do DNS** do balanceador para uso posterior.

---

## Tarefa 3: Criar um Modelo de Execução
1. No Console EC2, vá até **Modelos de execução** e selecione **Criar modelo de execução**.
2. Configure:
   - Nome: `lab-app-launch-template`
   - AMI: `WebServerAMI`
   - Tipo de instância: `t3.micro`
   - Grupo de segurança: `Grupo de segurança da web`
3. Selecione **Criar modelo de execução** e confirme a criação.

---

## Tarefa 4: Criar um Grupo do Auto Scaling
1. No Console EC2, selecione `lab-app-launch-template` e crie um **Grupo do Auto Scaling**.
2. Configure:
   - Nome: `Lab Auto Scaling Group`
   - VPC: `VPC do laboratório`
   - Sub-redes: `Sub-rede privada 1` e `Sub-rede privada 2`
3. Associe ao **Balanceador de Carga** e ao **Grupo de destino** (`lab-target-group`).
4. Defina a **Capacidade**:
   - Mínima: `2`
   - Máxima: `4`
   - Política de escalabilidade: `Utilização média da CPU 50%`
5. Finalize a criação.

---

## Tarefa 5: Verificar o Balanceamento de Carga
1. No Console EC2, vá até **Instâncias** e verifique se há duas novas instâncias.
2. Confirme a aprovação na **health check** do balanceador de carga.
3. Copie o **Nome do DNS** do balanceador e acesse no navegador para validar a aplicação.

---

## Tarefa 6: Testar o Auto Scaling
1. No Console AWS, acesse **CloudWatch > Alarmes**.
2. Aguarde o alarme `AlarmHigh` ser ativado.
3. No aplicativo **Teste de carga**, inicie a carga para aumentar a utilização da CPU.
4. Volte para **CloudWatch** e verifique o aumento de instâncias pelo Auto Scaling.

---

## Tarefa 7: Encerrar a Instância Web Server 1
1. No Console EC2, selecione **Web Server 1**.
2. No menu **Estado da instância**, escolha **Encerrar instância**.

---

## Desafio Opcional: Criar uma AMI utilizando a AWS CLI
1. Conecte-se a uma instância EC2 via **Amazon EC2 Instance Connect**.
2. Configure as credenciais da AWS CLI.
3. Execute o comando para criar uma AMI:
   ```sh
   aws ec2 create-image --instance-id <INSTANCE_ID> --name "MyCustomAMI" --no-reboot
   ```

---

## Conclusão
Parabéns! Você concluiu com êxito:
- Criação de uma **AMI**.
- Configuração de um **Balanceador de Carga**.
- Criação de um **Modelo de Execução e Grupo do Auto Scaling**.
- Monitoramento com **CloudWatch**.
- Teste de **Auto Scaling** para escalabilidade dinâmica.

