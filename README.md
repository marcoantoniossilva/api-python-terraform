# IFBA - Turma Pos Graduacao em Desenvolvimento Web - Disciplina Infraestrutura para Sistemas Web

# API CRUD com Python e Flask
Esta é uma API RESTful desenvolvida em Python utilizando o framework [Flask](https://flask.palletsprojects.com/), permitindo operações de CRUD (Create, Read, Update, Delete) em um banco de dados MySQL hospedado no Amazon RDS. A infraestrutura da aplicação é gerenciada como código com o [Terraform](https://www.terraform.io/), e o deploy é automatizado através do [GitHub Actions](https://docs.github.com/en/actions). 

A infraestrutura foi provisionada na AWS, utilizando uma instância Ubuntu do [EC2](https://docs.aws.amazon.com/ec2/index.html) para hospedar a aplicação Flask e o [RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.html) para o banco de dados MySQL. 

O processo de deploy da aplicação é realizado automaticamente com o GitHub Actions, integrando o código do repositório e realizando as operações de build e deployment na AWS.

Link de vídeo explicativo do processo no Youtube [Aqui](https://youtu.be/74djzDIFNjU)

# Documentação de Configuração

## 1. Configuração do AWS

### 1.1 Criar um Usuário

- Acesse o console da AWS e crie um usuário do IAM (não precisa de acesso ao console).
- Crie um grupo com a política "AdministratorAccess" e associe o usuário a este grupo.

### 1.2 Armazenar Credenciais

- Após criar o usuário, crie uma cheva de acesso para ele e guarde as credenciais:
  - **Access Key ou Chave de acesso**
  - **Secret Key ou Chave de acesso secreta**

### 1.3 Criar Chave Privada

- No serviço EC2, Crie uma chave privada (Pares de chaves) no formato RCA (.pem), baixe-a e coloque-a na pasta iac/.

### 1.4 Configurar chave

- Abra o arquivo iac/main.tf e substitua o valor do parâmetro "key_name" do recurso aws_instance.web para ficar com o mesmo nome da chave criada no passo anterior.

---

## 2. Configuração do Terraform

### 2.1 Instalar o Terraform

  - Execute os seguintes comandos para instalar o Terraform CLI:
    ```console
        sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
        sudo apt-get install terraform
        
        wget -O- https://apt.releases.hashicorp.com/gpg | \
        gpg --dearmor | \
        
        sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

        echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
        https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
        sudo tee /etc/apt/sources.list.d/hashicorp.list

        sudo apt update
        sudo apt-get install terraform
    ```

### 2.2 Inicializar o Terraform

- Navegue até a pasta `/iac` e inicialize o Terraform com o seguinte comando:
  ```console
    terraform init
  ```

---

## 3. Configuração do cliente AWS

### 3.1 Instale o aws client (Linux)
    
- Execute os seguintes comandos para instalar o AWS CLI:
    ```console
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        unzip awscliv2.zip
        sudo ./aws/install
    ```
    
### 3.2 Configure as credenciais do aws provider

- Configure as credenciais do AWS provider com o seguinte comando:
    ```console
        aws configure
    ```

Para a região, pode-se usar a "us-east-1" ou outra qualquer.

### 3.3 Forneça o access_key e a secret_key criados no passo 1.2

- Insira o Access Key e a Secret Key criados no passo 1.2.

---

## 4. Efetue o deploy da infraestrutura como código com o terraform

- Entre na pasta `/iac` e execute o terraform:
    ```console
        terraform apply
    ```

---

## 5. Configurar o banco
### 5.1 Configure o nível de segurança para a chave criada no passo 1.3
  ```console
    sudo chmod 600 {caminho para a chave criada no passo 1.3}
  ```
Se estiver utilizando o linux via WSL (Subsistema Windows para Linux), pode ser necessário copiar a chave para um diretório gerenciado pelo Linux, como /home/ por exemplo.

### 5.2 Obtenha o ipv4 público da instância E2C com o comando:
  ```console
    aws ec2 describe-instances --filters "Name=tag:Name,Values=HelloWorld2" --query "Reservations[*].Instances[*].[InstanceId, PublicIpAddress]" --output table
  ```
### 5.3 Obtenha o endpoint da instância RDS com o comando:
  ```console
    aws rds describe-db-instances --query "DBInstances[*].[DBInstanceIdentifier, Endpoint.Address]" --output table
  ```
### 5.4 Conecte via SSH na instância E2C
  ```console
    ssh -i {caminho para a chave criada no passo 1.3} ubuntu@{ipv4 público da instância}
  ```
### 5.5 Após conectar-se à instância EC2, conecte-se ao banco:
  ```console
    mysql -h {endpoint público da instância RDS} -u myapp_user -pmyapp_passwd myapp
  ```
  OBS: Note que não há espaço entre o parâmetro "-p" e a senha.

### 5.6 Insira os dados

 Após conectar-se ao banco, copie o conteúdo do arquivo db/db.sql e cole no console do mysql

- Saia do banco
  ```console
    exit
  ```

  ---

## 7. Configurando repositório

### 7.1 Configure o repositório e faça o push do código para o github
  ```console
    git init
    git add .
    git commit -m "Initial commit"
    git branch -M main
    git remote add origin {link para seu git repo}
    git push -u origin main
  ```

### 7.2 Adicione as seguintes variáveis de ambiente no GitHub Secrets:

  Seu repositório -> Settings -> Secrets and variables -> Actions -> New repository secret
  **Name:** "KEY"
  **Secret:** Conteúdo da chave privada RCA (.pem)

  O conteúdo pode ser obtido abrindo o arquivo em um editor de textos ou digitando no terminal:
    ```console
          cat {caminho para seu arquivo .pem}
    ```
  **Name:** "USERNAME"
  **Secret:** ubuntu

  **Name:** "HOST"
  **Secret:** Endereço ipv4 público da instância EC2 obtido no passo 5.2

  **Name:** "DB_USERNAME"
  **Secret:** myapp_user

  **Name:** "DB_PASSWORD"
  **Secret:** myapp_passwd

  **Name:** "DB_HOST"
  **Secret:** Endpoint público da instância RDS obtido no passo 5.3

  **Name:** "DB_NAME"
  **Secret:** myapp

---

## 8. Deploy e teste

- Efetue um push na main ou dispare o deploy em GitHub Actions

- Conecte na instancia EC2 (repita o passo 5.4)

- Inicie o flask com os seguintes comandos:
  ```console
    python3 -m venv /home/ubuntu/
    source /home/ubuntu/bin/activate
    flask --app /home/ubuntu/myapp/myapi run --host=0.0.0.0
  ```
  OBS: Caso dê erro de flask, tente instalar as bibliotecas do python manualmente:
  ```console
    pip install flask flask-mysqldb flask-cors
  ```

- Digite o endereço ipv4 público da instância EC2 obtido no passo 5.2 no seu navegador e teste a aplicação
