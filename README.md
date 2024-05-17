# Lançando uma instancia na AWS com Terraform 

# 1 Instalando Terraform 

- Windows 

- Instalar o gerenciador de pacotes chocolatey
- Entre no powershell no modo ADMINISTRADOR e execute o comando 
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```
- Depois execute o comando 
```
choco install terraform
```
- Linux 

- Execute os comandos 
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# 2 Criado um arquivo main.tf

# 3 Criando um profile na AWS (IAM) e se conectando a ele


- Criar as credencias CLI para este usuário
- Ir terminal visual code e coloca as credenciais
```
AWS configure --profile "profile criado" ex: bia-tf
```
- Adicionar o profile no arquivo terraform

```
terraform init
```

# 4 Criando SG na AWS

- Nome bia-dev
- Regras padrões do SG

# 5 Criando uma IAM Role Acesso SSM

- Permissões da role AmazonSSMManagedEC2InstanceDefaultPolicy
- Nome da role: role-acesso-ssm

# 6 Criando instância Manual e colocando SG e IAM role

# 7 Lançando instância pelo AWS cloudshell

- Entre no cloudshell 
```
git clone https://github.com/henrylle/bia.git
```
- cd bia
- cd scripts
- ./lancar_ec2_zona_a.sh 

- Mudar o nome da instância criada para bia-cli
- OBS: essa instância foi criada por linha de comando, podendo ser alterada os qualquer característica das instância

# 8 instale as 2 extensões do terraform no visual code

# 9 Criando um recurso no terraform para lançar a instância

- adicione um arquivo userdata.sh e coloque os comandos nele

```
#!/bin/bash

#Instalar Docker e Git
sudo yum update -y
sudo yum install git -y
sudo yum install docker -y
sudo usermod -a -G docker ec2-user
sudo usermod -a -G docker ssm-user
id ec2-user ssm-user
sudo newgrp docker

#Ativar docker
sudo systemctl enable docker.service
sudo systemctl start docker.service

#Instalar docker compose 2
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/download/v2.23.3/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose


#Adicionar swap
sudo dd if=/dev/zero of=/swapfile bs=128M count=32
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo echo "/swapfile swap swap defaults 0 0" >> /etc/fstab


#Instalar node e npm
curl -fsSL https://rpm.nodesource.com/setup_21.x | sudo bash -
sudo yum install -y nodejs
```
- Acrescente agora os comandos para criar a instância na AWS pelo terraform 

```
resource "aws-instance" "bia-dev" {
  ami = "ami-02f3f602d23f1659d"
  instance_type = "t3.micro"
  tags = {
    Name = "bia-terraform"
  }
  vpc_security_group_ids = ["sg-0aec4b0c06aaa5c5d"]
  user_data = file("userdata.sh")
  iam_instance_profile = "role-acesso-ssm"
}
```

```
terraform plan
```

```
terraform apply
```

# 10 Instale o plugin SSM

# 11 Acessando a instância via ssm
```
aws ssm start-session --target i-0e76f7270d6811857 --profile bia-tf
```
- target = instância que precisa acessar 
- profile = perfil que criou para ter acesso a instância

```
sudo su ec2-user
```

```
cd /home/ec2-user/
```

- Teste para ver se instalou os programas como o Docker e outros

# 12 Clonando o repositório e dentro da instância e rodando docker-compose

```
sudo su ec2-user
```

```
cd /home/ec2-user/
```
git clone https://github.com/henrylle/bia.git

- cd bia
- docker compose up -d 
