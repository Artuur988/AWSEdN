# Criar um site no S3

Este documento descreve os passos para iniciar e configurar instâncias EC2 na AWS, utilizando o Console de Gerenciamento da AWS e a AWS CLI.

## Objetivos 
- Executar comandos da AWS CLI que usam os serviços do IAM e do Amazon S3.

- Implantar um site estático em um bucket do S3.

- Criar um script que use a AWS CLI para copiar arquivos de um diretório local para o Amazon S3.

## Diagrama do fluxo 

![Criar um Site S3 drawio](https://github.com/user-attachments/assets/78701bd8-1611-4fe4-9e87-cd0bd2f09800)
---

# AWS S3 Setup and Management Using AWS CLI

## Tarefa 1: Conectar-se a uma instância do Amazon Linux EC2 usando SSM
1. No console da AWS, selecione o botão **Detalhes** na parte superior e escolha **Mostrar**.
2. Copie o valor `InstanceSessionUrl` da lista e cole-o em uma nova guia do navegador.
3. A conexão será estabelecida com `ssm-user` e um prompt será exibido.
4. Para mudar para o usuário `ec2-user` e acessar o diretório inicial, execute os seguintes comandos:
   ```sh
   sudo su -l ec2-user
   pwd
   ```

## Tarefa 2: Configurar a AWS CLI
1. Execute o comando abaixo para iniciar a configuração da AWS CLI:
   ```sh
   aws configure
   ```
2. No prompt, forneça as seguintes informações:
   - **ID de chave de acesso da AWS**: copie e cole o valor `AccessKey` do painel esquerdo.
   - **AWS Secret Access Key**: copie e cole o valor `SecretKey` do painel esquerdo.
   - **Nome da região padrão**: insira `us-west-2`.
   - **Formato de saída padrão**: insira `json`.

## Tarefa 3: Criar um bucket do S3 usando a AWS CLI
1. Para criar um bucket do S3, execute o seguinte comando, substituindo `<nome-do-bucket>` por um nome único:
   ```sh
   aws s3api create-bucket --bucket <nome-do-bucket> --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2
   ```
2. Se o comando for bem-sucedido, a saída exibirá o nome do bucket criado.

## Tarefa 4: Criar um usuário do IAM com acesso ao Amazon S3
1. Criar um usuário IAM chamado `awsS3user`:
   ```sh
   aws iam create-user --user-name awsS3user
   ```
2. Criar um perfil de login para o usuário:
   ```sh
   aws iam create-login-profile --user-name awsS3user --password Training123!
   ```
3. Obtenha o **ID da conta** da AWS no Console de Gerenciamento da AWS.
4. Anexar a política gerenciada da AWS para conceder acesso total ao Amazon S3 ao usuário `awsS3user`:
   ```sh
   aws iam list-policies --query "Policies[?contains(PolicyName,'S3')]"
   aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/<policyYouFound> --user-name awsS3user
   ```

## Tarefa 5: Ajustar permissões do bucket S3
1. No Console de Gerenciamento da AWS, acesse o console do Amazon S3.
2. Selecione o nome do bucket criado e vá até a aba **Permissões**.
3. Em **Bloquear acesso público (configurações do bucket)**, selecione **Editar** e desmarque a opção **Bloquear todo o acesso público**.
4. Clique em **Salvar alterações** e confirme a ação.
5. Em **Propriedade do objeto**, ative **ACLs habilitadas**, confirme a ação e clique em **Salvar alterações**.

## Tarefa 6: Extrair os arquivos necessários
1. Execute os seguintes comandos no terminal SSH para extrair os arquivos:
   ```sh
   cd ~/sysops-activity-files
   tar xvzf static-website-v2.tar.gz
   cd static-website
   ```
2. Verifique se os arquivos foram extraídos corretamente:
   ```sh
   ls
   ```
   Deve listar `index.html` e diretórios como `css` e `imagens`.

## Tarefa 7: Fazer upload de arquivos para o Amazon S3
1. Ativar a hospedagem de site no bucket:
   ```sh
   aws s3 website s3://<my-bucket>/ --index-document index.html
   ```
2. Fazer upload dos arquivos para o bucket:
   ```sh
   aws s3 cp /home/ec2-user/sysops-activity-files/static-website/ s3://<my-bucket>/ --recursive --acl public-read
   ```
3. Verificar o upload dos arquivos:
   ```sh
   aws s3 ls s3://<my-bucket>/
   ```

## Tarefa 8: Criar um script para atualização do site
1. Criar um script de atualização:
   ```sh
   cd ~
   touch update-website.sh
   vi update-website.sh
   ```
2. No editor VI, pressione `i` para entrar no modo de edição e adicione:
   ```sh
   #!/bin/bash
   aws s3 cp /home/ec2-user/sysops-activity-files/static-website/ s3://<my-bucket>/ --recursive --acl public-read
   ```
   Salve e saia com `:wq`.
3. Tornar o script executável:
   ```sh
   chmod +x update-website.sh
   ```
4. Atualizar o site executando o script:
   ```sh
   ./update-website.sh
   ```

## Desafio Opcional: Usar `aws s3 sync`
1. Substituir `aws s3 cp` por `aws s3 sync` para otimizar o upload:
   ```sh
   aws s3 sync /home/ec2-user/sysops-activity-files/static-website/ s3://<my-bucket>/ --acl public-read
   ```
2. Atualize o site no navegador para verificar as mudanças.

## Conclusão
- Executou comandos da AWS CLI para IAM e Amazon S3.
- Criou um bucket do Amazon S3 e configurou permissões.
- Implantou um site estático no S3.
- Criou um script para facilitar atualizações futuras do site.
