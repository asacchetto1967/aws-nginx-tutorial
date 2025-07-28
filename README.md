Criando um Servidor Nginx Simples na AWS com Cloud Shell
Este tutorial detalha como configurar uma infraestrutura básica na Amazon Web Services (AWS) usando o AWS Cloud Shell para lançar um servidor web Nginx. Ao final, você terá um par de chaves SSH, uma VPC, uma sub-rede, um grupo de segurança com regras de acesso (SSH na porta 22 e HTTP na porta 80), um Internet Gateway para conectividade externa, e uma instância EC2 executando Nginx com uma página web personalizada.


ATENÇÃO: ESTE TUTORIAL NÃO É PARA AMBIENTES DE PRODUÇÃO. Ele foi projetado para fins educacionais e não incorpora as melhores práticas de segurança, deixando a instância e os serviços extremamente vulneráveis.

Pré-requisitos
Uma conta AWS ativa.

Acesso ao AWS Cloud Shell (diretamente no console da AWS).

Passo a Passo
1. Criar Par de Chaves para Acesso SSH
O par de chaves permite a comunicação segura com sua instância EC2 via SSH.

Comando:

Bash

aws ec2 create-key-pair --key-name escola-da-nuvem-key --query 'KeyMaterial' --output text > escola-da-nuvem-key.pem
Observações:

Este comando cria a chave privada e a salva diretamente no arquivo 

escola-da-nuvem-key.pem.

1.1. Alterar as Permissões do Arquivo
É crucial definir as permissões corretas para sua chave privada, pois o SSH exige que ela seja acessível apenas pelo proprietário do arquivo.

Comando:

Bash

chmod 400 escola-da-nuvem-key.pem
1.2. Fazer Download da Chave Privada (Opcional, para acesso externo)
Se você planeja acessar a instância SSH do seu computador local (fora do Cloud Shell), precisará baixar o arquivo .pem.

Passos no Cloud Shell:

No canto superior direito do Cloud Shell, clique em 

Actions.

Selecione 

Download File.

Na janela que abrir, digite o caminho para sua chave: 

/home/cloudshell-user/escola-da-nuvem-key.pem.

Clique em 

Download.

2. Criar a VPC (Virtual Private Cloud)
A VPC é uma rede virtual privada e isolada na nuvem da AWS para seu projeto.

Comando:

Bash

VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text)
Observações:

O 

VPC_ID=$(...) executa o comando e atribui o resultado à variável VPC_ID.


--cidr-block 10.0.0.0/16 define o bloco de endereços IP da sua VPC, oferecendo 65.536 endereços possíveis (menos 5 IPs reservados pela AWS por sub-rede).



2.1. Adicionar um Nome à VPC para Fácil Identificação
Comando:

Bash

aws ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=EscolaDaNuvemVPC
2.2. Listar as Informações Sobre a Sua VPC
Comando:

Bash

aws ec2 describe-vpcs --filters "Name=tag:Name,Values=EscolaDaNuvemVPC"
3. Criar Sub-rede
Uma sub-rede é um segmento da sua VPC.

Comando:

Bash

SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 --query Subnet.SubnetId --output text)
Observações:


--vpc-id $VPC_ID associa a sub-rede à sua VPC.



--cidr-block 10.0.1.0/24 define o bloco de endereços para esta sub-rede, com 256 endereços possíveis (menos 5 IPs reservados).


3.1. Adicionar um Nome à Sub-rede para Fácil Identificação
Comando:

Bash

aws ec2 create-tags --resources $SUBNET_ID --tags Key=Name,Value=EscolaDaNuvemSubnet
echo "Sub-rede Criada com ID: $SUBNET_ID"
4. Criar um Internet Gateway
Um Internet Gateway permite que seu servidor web seja acessado através da internet.

Comando:

Bash

IGW_ID=$(aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text)
4.1. Adicionar um Nome ao Internet Gateway para Fácil Identificação
Comando:

Bash

aws ec2 create-tags --resources $IGW_ID --tags Key=Name,Value=EscolaDaNuvemIGW
4.2. Anexar o Internet Gateway à VPC
Comando:

Bash

aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID
echo "Internet Gateway Criado com ID: $IGW_ID e anexado à VPC"
5. Criar uma Tabela de Rotas
Uma tabela de rotas contém regras que controlam para onde o tráfego da rede é direcionado.

Comando:

Bash

ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query RouteTable.RouteTableId --output text)
5.1. Adicionar um Nome à Tabela de Rotas para Fácil Identificação
Comando:

Bash

aws ec2 create-tags --resources $ROUTE_TABLE_ID --tags Key=Name,Value=EscolaDaNuvemRouteTable
5.2. Criar uma Rota para a Internet
Esta rota direciona todo o tráfego de saída (não destinado à VPC) para o Internet Gateway.

Comando:

Bash

aws ec2 create-route --route-table-id $ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
5.3. Associar a Tabela de Rotas à Sub-rede
A sub-rede agora usará esta tabela de rotas para o tráfego de saída.


Comando:

Bash

aws ec2 associate-route-table --subnet-id $SUBNET_ID --route-table-id $ROUTE_TABLE_ID
6. Criar um Grupo de Segurança
O Security Group atua como um firewall virtual para sua instância EC2, controlando o tráfego de entrada e saída.


Comando:

Bash

SG_ID=$(aws ec2 create-security-group --group-name EscolaDaNuvemSG --description "Acesso SSH e HTTP" --vpc-id $VPC_ID --query GroupId --output text)
6.1. Adicionar um Nome ao Grupo de Segurança para Fácil Identificação
Comando:

Bash

aws ec2 create-tags --resources $SG_ID --tags Key=Name,Value=EscolaDaNuvemSG
echo "Grupo de Segurança Criado com ID: $SG_ID"
6.2. Liberar o Acesso SSH (porta 22) e HTTP (porta 80)
Comandos:

Bash

MY_IP=$(curl -s http://checkip.amazonaws.com)/32
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 22 --cidr $MY_IP
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0
Observações:


MY_IP=$(curl -s http://checkip.amazonaws.com)/32 descobre seu IP público atual para restringir o acesso SSH apenas a ele.


--cidr 0.0.0.0/0 para HTTP permite o acesso de qualquer IPV4, o que é necessário para um servidor web público. Lembre-se que seu IP público pode mudar.


7. Lançar a Instância EC2 Amazon Linux
7.1. Obter o ID da AMI (Amazon Machine Image)
Usaremos a AMI mais recente do Amazon Linux 2 para esta instância.

Comando:

Bash

AMI_ID=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].Value' --output text)
echo "Utilizando a AMI ID: $AMI_ID"
7.2. Criar a Instância EC2
Este comando lança sua instância EC2, associando todos os recursos de rede e segurança que você criou.

Comando:

Bash

INSTANCE_ID=$(aws ec2 run-instances \
    --image-id $AMI_ID \
    --instance-type t2.micro \
    --key-name escola-da-nuvem-key \
    --security-group-ids $SG_ID \
    --subnet-id $SUBNET_ID \
    --associate-public-ip-address \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=EscolaDaNuvem-WebApp}]' \
    --query 'Instances[0].InstanceId' --output text)
Aguardar a Instância em Execução:

Bash

echo "Aguardando a instância com ID: $INSTANCE_ID estar em execução..."
aws ec2 wait instance-running --instance-ids $INSTANCE_ID
echo "Instância em execução!"
Observações:


--associate-public-ip-address atribui um IP público à instância, tornando-a acessível pela internet.

7.3. Obter o Endereço IP Público da Instância
Você precisará deste IP para acessar sua instância via SSH e pelo navegador.

Comando:

Bash

PUBLIC_IP=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)
echo "O endereço IP público da sua instância é: $PUBLIC_IP"
8. Instalando e Configurando o Servidor Web Nginx
Agora você irá se conectar à sua instância e instalar o Nginx.

8.1. Conectar à Instância Via SSH
Comando:

Bash

ssh -i "escola-da-nuvem-key.pem" ec2-user@$PUBLIC_IP
Observações:


ssh -i "escola-da-nuvem-key.pem" usa a chave que você baixou.


ec2-user é o nome de usuário padrão para instâncias Amazon Linux.


$PUBLIC_IP é o endereço IP que você obteve no passo anterior.

8.2. Dentro da Instância, Atualizar os Pacotes e Instalar o Nginx
Após conectar-se via SSH:

Bash

sudo yum update -y
sudo amazon-linux-extras install nginx1.12 -y
Observações:


sudo yum update -y atualiza os pacotes do sistema.


sudo amazon-linux-extras install nginx1.12 -y instala a versão 1.12 do Nginx.

8.3. Iniciar o Serviço do Nginx e Habilitá-lo para Iniciar com o Sistema
Bash

sudo systemctl start nginx
sudo systemctl enable nginx
Observações:


sudo systemctl start nginx inicia o serviço Nginx.


sudo systemctl enable nginx configura o Nginx para iniciar automaticamente a cada boot da instância.

8.4. Criar a Página Web Personalizada
Você irá substituir a página index.html padrão do Nginx.

8.4.1. Acessar o Diretório Raiz do Nginx
Bash

cd /usr/share/nginx/html
8.4.2. Criar ou Editar o Arquivo index.html
Bash

sudo nano index.html
Cole o conteúdo HTML e CSS abaixo no editor nano:

HTML

<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>REPOSITÓRIO DE LINKS UTEIS CURSO AWS DEVELOPER ASSOCIATE ESCOLA DA NUVEM</title>
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
<style>
body {
font-family: 'Roboto', sans-serif; /* Fonte moderna */
background-color: #ADD8E6; /* Azul claro */
color: #333;
margin: 0;
padding: 20px;
display: flex;
flex-direction: column;
align-items: center;
justify-content: center;
min-height: 100vh;
box-sizing: border-box;
}
.container {
background-color: #fff;
padding: 30px 50px;
border-radius: 10px;
box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
max-width: 800px;
width: 100%;
}
h1 {
color: #0056b3;
text-align: center;
margin-bottom: 30px;
font-size: 2.2em;
}
ul {
list-style: none;
padding: 0;
}
li {
margin-bottom: 15px;
}
a {
color: #007bff;
text-decoration: none;
font-weight: bold;
font-size: 1.1em;
transition: color 0.3s ease;
}
a:hover {
color: #0056b3;
text-decoration: underline;
}
.footer {
margin-top: 40px;
font-size: 0.9em;
color: #555;
}
</style>
</head>
<body>
<div class="container">
<h1>REPOSITÓRIO DE LINKS UTEIS CURSO AWS DEVELOPER ASSOCIATE ESCOLA DA NUVEM</h1>
<ul>
<li><a href="https://aws.amazon.com/certification/certified-developer-associate/" target="_blank">Certificação AWS Certified Developer - Associate (Página Oficial AWS)</a></li>
<li><a href="https://www.examprepper.co/exam/25/1" target="_blank">Exam Prepper - AWS Developer Associate Practice Exams</a></li>
<li><a href="https://www.youtube.com/playlist?list=PLK2b5y9F1DqZRzd5x69Z4ndR7jsPj7ghr" target="_blank">Conteúdo do Youtube via Google User Content</a></li>
</ul>
<p class="footer">Criado para o Curso AWS Developer Associate</p>
</div>
</body>
</html>
Após colar o conteúdo no nano:

Pressione 

Ctrl + X para sair.

Pressione 

Y para confirmar que deseja salvar as alterações.

Pressione 

Enter para confirmar o nome do arquivo (index.html).

8.4.3. Verificar as Permissões do Arquivo
Comando:

Bash

sudo chmod 644 index.html
Observações:

sudo chmod 644 index.html garante que o Nginx tenha permissão para ler seu novo arquivo. As permissões 

644 são ideais para arquivos web (leitura e escrita para o proprietário, apenas leitura para o grupo e outros).

8.4.4. Reiniciar o Nginx
Bash

sudo systemctl restart nginx
Observações:

Este comando reinicia o Nginx para que ele comece a servir o novo conteúdo.


9. Testar o Servidor Nginx
Abra um navegador e digite o endereço IP público da sua instância ($PUBLIC_IP). Você deverá ver a página personalizada que acabou de criar.

Limpando a Infraestrutura
É extremamente importante deletar os recursos criados após concluir seus estudos para evitar cobranças indesejadas na sua conta AWS. Este script assume que você está na mesma sessão do Cloud Shell onde os comandos de criação foram executados, e as variáveis ($VPC_ID, $SUBNET_ID, etc.) ainda estão definidas.

Bash

#!/bin/bash

echo "Iniciando a exclusão da infraestrutura AWS usando variáveis da sessão..."

# **Verificação de Variáveis (Opcional, mas recomendado para segurança)**
# Se alguma variável não estiver definida, o comando correspondente será ignorado.
echo "IDs de Recursos a serem deletados:"
echo "Instance ID: ${INSTANCE_ID:-N/A}"
echo "Security Group ID: ${SG_ID:-N/A}"
echo "Route Table ID: ${ROUTE_TABLE_ID:-N/A}"
echo "Internet Gateway ID: ${IGW_ID:-N/A}"
echo "Subnet ID: ${SUBNET_ID:-N/A}"
echo "VPC ID: ${VPC_ID:-N/A}"
echo "--------------------------------------------------"


# 1. Terminar a Instância EC2
if [ -n "$INSTANCE_ID" ]; then
    echo "1. Terminando instância EC2: $INSTANCE_ID..."
    aws ec2 terminate-instances --instance-ids "$INSTANCE_ID"
    echo "Aguardando a instância terminar. Isso pode levar alguns minutos..."
    aws ec2 wait instance-terminated --instance-ids "$INSTANCE_ID"
    echo "Instância $INSTANCE_ID terminada."
else
    echo "Variável INSTANCE_ID não definida. Pulando término da instância."
fi

# 2. Deletar o Security Group
if [ -n "$SG_ID" ]; then
    echo "2. Deletando Security Group: $SG_ID..."
    aws ec2 delete-security-group --group-id "$SG_ID"
    echo "Security Group $SG_ID deletado."
else
    echo "Variável SG_ID não definida. Pulando exclusão do Security Group."
fi

# 3. Remover a rota para o Internet Gateway da Tabela de Rotas
if [ -n "$ROUTE_TABLE_ID" ] && [ -n "$IGW_ID" ]; then
    echo "3. Deletando rota 0.0.0.0/0 para $IGW_ID na tabela de rotas: $ROUTE_TABLE_ID..."
    aws ec2 delete-route --route-table-id "$ROUTE_TABLE_ID" --destination-cidr-block 0.0.0.0/0
    echo "Rota 0.0.0.0/0 removida de $ROUTE_TABLE_ID."
else
    echo "Variáveis ROUTE_TABLE_ID ou IGW_ID não definidas. Pulando exclusão da rota."
fi

# 4. Deletar a Tabela de Rotas Personalizada
if [ -n "$ROUTE_TABLE_ID" ]; then
    echo "4. Deletando Tabela de Rotas: $ROUTE_TABLE_ID..."
    aws ec2 delete-route-table --route-table-id "$ROUTE_TABLE_ID"
    echo "Tabela de Rotas $ROUTE_TABLE_ID deletada."
else
    echo "Variável ROUTE_TABLE_ID não definida. Pulando exclusão da Tabela de Rotas."
fi

# 5. Desanexar e Deletar o Internet Gateway
if [ -n "$IGW_ID" ] && [ -n "$VPC_ID" ]; then
    echo "5. Desanexando Internet Gateway: $IGW_ID do VPC: $VPC_ID..."
    aws ec2 detach-internet-gateway --internet-gateway-id "$IGW_ID" --vpc-id "$VPC_ID"
    echo "Internet Gateway $IGW_ID desanexado."

    echo "Deletando Internet Gateway: $IGW_ID..."
    aws ec2 delete-internet-gateway --internet-gateway-id "$IGW_ID"
    echo "Internet Gateway $IGW_ID deletado."
else
    echo "Variáveis IGW_ID ou VPC_ID não definidas. Pulando exclusão do Internet Gateway."
fi

# 6. Deletar a Sub-rede
if [ -n "$SUBNET_ID" ]; then
    echo "6. Deletando Sub-rede: $SUBNET_ID..."
    aws ec2 delete-subnet --subnet-id "$SUBNET_ID"
    echo "Sub-rede $SUBNET_ID deletada."
else
    echo "Variável SUBNET_ID não definida. Pulando exclusão da Sub-rede."
fi

# 7. Deletar o VPC
if [ -n "$VPC_ID" ]; then
    echo "7. Deletando VPC: $VPC_ID..."
    aws ec2 delete-vpc --vpc-id "$VPC_ID"
    echo "VPC $VPC_ID deletado."
else
    echo "Variável VPC_ID não definida. Pulando exclusão do VPC."
fi

echo "Processo de exclusão de infraestrutura concluído (ou iniciado para recursos dependentes de tempo)."
echo "Verifique o console da AWS para confirmar a remoção de todos os recursos."