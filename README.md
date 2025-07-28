VERSÃO PT-BR
Criando um Servidor Nginx Simples na AWS com Cloud Shell
DESCRIÇÃO: vamos criar uma estrutura básica na AWS, no fim teremos criado nosso par de chaves, a VPC, a sub-rede, o grupo de segurança com as liberações de acesso via SSH na porta 22 e acesso HTTP na porta 80, um Internet Gateway para acesso à internet e um servidor web Nginx instalado e exibindo uma mensagem personalizada, acredito que é interessante que cada um altere os nomes das chaves, redes e coloque a sua própria mensagem e até mesmo imagens e links na página do servidor caso deseje.

NÃO UTILIZE ESSE TUTORIAL PARA PRODUÇÃO, ELE NÃO CONTÉM PRÁTICAS RECOMENDÁVEIS DE SEGURANÇA E A INSTÂNCIA E SERVIÇOS FICAM EXTREMAMENTE VULNERÁVEIS!

Pré-requisitos
Uma conta AWS ativa.

Acesso ao AWS Cloud Shell (diretamente no console da AWS).

Passo a Passo
1-) CRIAR PAR DE CHAVES PARA ACESSO SSH:

COMANDO:

Bash

aws ec2 create-key-pair --key-name escola-da-nuvem-key --query 'KeyMaterial' --output text > escola-da-nuvem-key.pem

OBSERVAÇÕES:
O comando aws ec2 create-key-pair cria a chave privada que irá permitir a comunicação segura com a instância EC2 através do protocolo SSH.
O comando --query 'KeyMaterial' --output text filtra a saída que seria em formato JSON para formato simples, não mostra o conteúdo e salva o arquivo.

1.1-) ALTERAR AS PERMISSÕES DO ARQUIVO:
COMANDO:

Bash

chmod 400 escola-da-nuvem-key.pem

1.2-) FAZER DOWNLOAD PARA SEU COMPUTADOR DA CHAVE PRIVADA:
Canto superior direito do Cloud Shell clique em ACTIONS

Selecione Download File

Na janela que abrir digite o caminho para sua chave
/home/cloudshell-user/escola-da-nuvem-key.pem

Clique em Download

2-) CRIAR A VPC (Virtual Private Cloud):

COMANDO:

Bash

VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text)

OBSERVAÇÕES:
O comando VPC_ID=$(...) executa tudo o que está dentro dos parênteses e atribui o resultado à variável chamada VPC_ID.
O comando aws ec2 create-vpc cria uma nova rede virtual privada e isolada na nuvem da AWS para seu projeto.
O comando --cidr-block 10.0.0.0/16 define o bloco de endereços privados que sua rede utilizará, a máscara /16 fornece 65.536¹ endereços IP possíveis, a definição do tamanho da rede dentro da VPC deve ser estudada cuidadosamente pois se for demasiado grande pode ter problemas de superposição quando em comunicação com outra VPC e se for pequena pode ficar sem endereço para suas instâncias.
O comando --query Vpc.VpcID –output text filtra a saída que seria em formato JSON para formato simples, não mostra o conteúdo e salva o arquivo para ser usado nos próximos comandos na mesma sessão.

2.1-) ADICIONAR UM NOME À VPC PARA FÁCIL IDENTIFICAÇÃO:
COMANDO:

Bash

aws ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=EscolaDaNuvemVPC

OBSERVAÇÕES:
O comando aws ec2 create-tags atribui metadados (tags) ao recurso escolhido na EC2.
O comando --resources $VPC_ID determina qual o recurso receberá o metadado.
O comando --tags Key=Name,Value=EscolaDaNuvemVPC define a tag.

2.2-) LISTAR AS INFORMAÇÕES SOBRE A SUA VPC:
COMANDO:

Bash

aws ec2 describe-vpcs --filters "Name=tag:Name,Values=EscolaDaNuvemVPC"

OBSERVAÇÕES:
O comando aws ec2 --describe-vpcs descreve todas as suas VPCs.
O comando --filters “Name=tag:Name,Values=Escola DaNuvemVPC” filtra pelo nome (tag) que foi atribuído no comando anterior.

3-) CRIAR SUB-REDE:

COMANDO:

Bash

SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 --query Subnet.SubnetId --output text)

OBSERVAÇÕES:
O comando SUBNET_ID=$(...) executa tudo o que está dentro dos parênteses e atribui o resultado à variável chamada SUBNET_ID.
O comando aws ec2 --create subnet cria uma nova sub-rede.
O comando --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 cria a sub-rede dentro da sua VPC e define o bloco de endereços privados que sua sub-rede utilizará, a máscara /24 fornece 256¹ endereços IP possíveis, a definição do tamanho da sub-rede dentro da VPC deve ser estudada cuidadosamente pois se for demasiado grande pode ter problemas de superposição quando em comunicação com outras sub-redes dentro de sua VPC e se for pequena pode ficar sem endereço para suas instâncias.
O comando --query Subnet.SubnetID --output text filtra a saída que seria em formato JSON para formato simples, não mostra o conteúdo e salva o arquivo para ser usado nos próximos comandos na mesma sessão.

3.1-) ADICIONE UM NOME À SUB-REDE PARA FÁCIL IDENTIFICAÇÃO:
COMANDO:

Bash

aws ec2 create-tags --resources $SUBNET_ID --tags Key=Name,Value=EscolaDaNuvemSubnet
echo “Sub-rede Criada com ID: $SUBNET_ID”

OBSERVAÇÕES:
O comando aws ec2 create-tags atribui metadados (tags) ao recurso escolhido na EC2.
O comando --resources $SUBNET_ID determina qual o recurso receberá o metadado.
O comando --tags Key=Name,Value=EscolaDaNuvemSubnet define a tag.
O comando echo “Sub-rede Criada com ID: $SUBNET_ID” imprime no terminal a frase “Sub-rede Criada com ID: seguida pelo valor real do ID da sub-rede que foi armazenado na variável SUBNET_ID.

4-) CRIAR UM INTERNET GATEWAY:

COMANDO:

Bash

IGW_ID=$(aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text)

OBSERVAÇÕES:
O comando IGW_ID=$(...) executa tudo o que está dentro dos parênteses e atribui o resultado à variável chamada IGW_ID.
O comando aws ec2 create-internet-gateway cria um Internet Gateway para que seu servidor web possa ser acessado através da internet.
O comando --query InternetGateway.InternetGatewayID --output text filtra a saída que seria em formato JSON para formato simples, não mostra o conteúdo e salva o arquivo para ser usado nos próximos comandos na mesma sessão.

4.1-) ADICIONE UM NOME AO INTERNET GATEWAY PARA FÁCIL IDENTIFICAÇÃO:
COMANDO:

Bash

aws ec2 create-tags --resources $IGW_ID --tags Key=Name,Value=EscolaDaNuvemIGW

OBSERVAÇÕES:
O comando aws ec2 create-tags atribui metadados (tags) ao recurso escolhido na EC2.
O comando --resources $IGW_ID determina qual o recurso receberá o metadado.
O comando --tags Key=Name,Value=EscolaDaNuvemSubnet define a tag.

4.2-) ANEXE O INTERNET GATEWAY À VPC:
COMANDO:

Bash

aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID

OBSERVAÇÕES:
O comando aws ec2 attach-internet-gateway anexa o internet gateway na sua VPC.
O comando --vpc-id $VPC_ID especifica qual é a VPC.
O comando --internet-gateway-id $IGW_ID especifica qual é o Internet Gateway.

COMANDO:

Bash

echo “Internet Gateway Criado com ID: $IGW_ID e anexado à VPC”

OBSERVAÇÕES:
O comando echo “Internet Gateway Criado com ID: $IGW_ID e anexado à VPC” imprime no terminal a frase “Internet Gateway Criado com ID: ” seguida pelo valor real do ID do Internet Gateway que foi armazenado na variável IGW_ID.

5-) CRIE UMA TABELA DE ROTAS:

COMANDO:

Bash

ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query RouteTableId --output text)

OBSERVAÇÕES:
O comando ROUTE TABLE ID=$(...) executa tudo o que está dentro dos parênteses e atribui o resultado à variável chamada ROUTE_TABLE_ID.
O comando aws ec2 create-route-table --vpc=id $VPC_ID cria uma tabela de rotas na VPC cujo ID foi armazenado na variável VPC_ID.
O comando --query RouteTableID --output text filtra a saída que seria em formato JSON para formato simples, não mostra o conteúdo e salva o arquivo para ser usado nos próximos comandos na mesma sessão.

5.1-) ADICIONE UM NOME A TABELA DE ROTAS PARA FÁCIL IDENTIFICAÇÃO:
COMANDO:

Bash

aws ec2 create-tags --resources $ROUTE_TABLE_ID --tags Key=Name,Value=EscolaDaNuvemRouteTable

OBSERVAÇÕES:
O comando aws ec2 create-tags atribui metadados (tags) ao recurso escolhido na EC2.
O comando --resources $ROUTE_TABLE_ID determina qual o recurso receberá o metadado.
O comando --tags Key=Name,Value=EscolaDaNuvemRouteTable define a tag.

5.2-) CRIE UMA ROTA PARA A INTERNET:
COMANDO:

Bash

aws ec2 create-route --route-table-id $ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID

OBSERVAÇÕES:
O comando aws ec2 create-route cria uma rota dentro da tabela de rotas com o ID armazenado na variável ROUTE_TABLE_ID.
O comando --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID está instruindo a tabela de rotas a enviar todo o tráfego que não seja para a própria VPC (ou para outras rotas específicas) para o Internet Gateway especificado pela variável IGW_ID.

5.3-) ASSOCIE A TABELA DE ROTAS A SUB-REDE:
COMANDO:

Bash

aws ec2 associate-route-table --subnet-id $SUBNET_ID --route-table-id $ROUTE_TABLE_ID

OBSERVAÇÕES:
O comando aws ec2 associate-route-table associa uma rota a uma sub-rede.
O comando --subnet-id $SUBNET_ID --route-table-id $ROUTE_TABLE_ID especifica qual sub-rede será associada com determinada rota.

6-) CRIE UM GRUPO DE SEGURANÇA:

COMANDO:

Bash

SG_ID=$(aws ec2 create-security-group --group-name EscolaDaNuvemSG --description "Acesso SSH e HTTP" --vpc-id $VPC_ID --query GroupId --output text)

OBSERVAÇÕES:
O comando SG_ID=$(...) executa tudo o que está dentro dos parênteses e atribui o resultado à variável chamada SG_ID.
O comando aws ec2 create-security-group cria o grupo de segurança.
O comando --group-name EscolaDaNuvemSG define o nome do seu grupo de segurança.
O comando --description “Acesso SSH e HTTP” descreve o grupo de segurança.
O comando --vpc-id $VPC_ID especifica que o grupo de segurança será criado na VPC cuja variável é VPC_ID.
O comando --query GroupId --output text filtra a saída que seria em formato JSON para formato simples, não mostra o conteúdo e salva o arquivo para ser usado nos próximos comandos na mesma sessão.

6.2-) ADICIONE UM NOME A TABELA DE ROTAS PARA FÁCIL IDENTIFICAÇÃO:
COMANDO:

Bash

aws ec2 create-tags --resources $SG_ID --tags Key=Name,Value=EscolaDaNuvemSG
echo "Grupo de Segurança Criado com ID: $SG_ID"

OBSERVAÇÕES:
O comando aws ec2 create-tags atribui metadados (tags) ao recurso escolhido na EC2.
O comando --resources $SG_ID determina qual o recurso receberá o metadado.
O comando --tags Key=Name,Value=EscolaDaNuvemSG define a tag.
O comando echo "Grupo de Segurança Criado com ID: $SG_ID imprime no terminal a frase "Grupo de Segurança Criado com ID:” seguido pelo valor real do ID do Internet Gateway que foi armazenado na variável SG_ID.

6.3-) LIBERE O ACESSO SSH (porta 22) E HTTP (porta 80):
COMANDO:

Bash

MY_IP=$(curl -s http://checkip.amazonaws.com)/32
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 22 --cidr $MY_IP

OBSERVAÇÕES:
O comando MY_IP=$(curl -s http://checkip.amazonaws.com)/32 utiliza a ferramenta curl e o serviço da AWS para descobrir seu IP público atual e armazenar na variável MY_IP.
O comando aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 22 --cidr $MY_IP adiciona uma regra de entrada (ingress rule) ao grupo de segurança cuja variável é SG_ID, permitindo que o tráfego SSH (porta 22) chegue a partir do seu IP atual².

COMANDO:

Bash

aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0

OBSERVAÇÕES:
O comando aws ec2 authorize-security-group-ingress --group-id $SG_ID adiciona uma regra de entrada (ingress rule) ao grupo de segurança cuja variável é SG_ID.
O comando --protocol tcp --port 80 --cidr 0.0.0.0/0 permite que o tráfego TCP (porta 80) chegue de qualquer endereço IPV4.

7-) LANCE A INSTÂNCIA EC2 AMAZON LINUX:

7.1-) OBTENHA O ID DA AMI:
COMANDO:

Bash

AMI_ID=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].Value' --output text)
echo "Utilizando a AMI ID: $AMI_ID"

OBSERVAÇÕES:
O comando AMI_ID=$(...) executa tudo o que está dentro dos parênteses e atribui o resultado à variável chamada AMI_ID.
O comando aws ssm get-parameters instrui o ssm (AWS System Manager) a extrair um valor da Parameter Store.
O comando --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 especifica o nome (caminho) do parâmetro que você deseja obter do Parameter Store, que sempre contém o ID da AMI mais recente do Amazon Linux 2.
O comando --query 'Parameters[0].Value' --output text filtra a saída que seria em formato JSON para formato simples, a expressão ‘Parameters[0].Value’ seleciona o primeiro item da lista e extrai o valor da ID da AMI, não mostra o conteúdo e salva o arquivo para ser usado nos próximos comandos na mesma sessão.
O comando echo "Utilizando a AMI ID: $AMI_ID" imprime no terminal a frase "Utilizando a AMI ID:” seguido pelo valor real da ID da AMI que foi armazenado na variável AMI_ID.

7.2-) CRIE A INSTÂNCIA EC2:
COMANDO:

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

AGUARDE ALGUNS INSTANTES E DEPOIS ENTRE COM O PRÓXIMO BLOCO:

COMANDO:

Bash

echo "Aguardando a instância com ID: $INSTANCE_ID estar em execução..."
aws ec2 wait instance-running --instance-ids $INSTANCE_ID
echo "Instância em execução!"
OBSERVAÇÕES:
Nessa sequência de comandos concatenados todas as variáveis criadas nos comandos anteriores são utilizadas, essa é uma prática comum e muito útil para ser usada na automação pois a infraestrutura na nuvem raramente é estática.
Cenários como múltiplos ambientes (desenvolvimento, teste, produção), diferentes regiões geográficas ou alterações de requisitos são comuns, bem como para reutilização e consistência do código e facilidade de manutenção.
O comando INSTANCE_ID=$(...) executa tudo o que está dentro dos parênteses e atribui o resultado à variável chamada INSTANCE_ID.
O comando aws ec2 run-instances lança uma ou mais (pode ser usado para lançar em lote) novas instâncias.
O comando --image-id $AMI_ID especifica qual AMI (Amazon Machine Image) será usado para criar a instância e a variável AMI_ID é a que foi criada no passo 7.1.
O comando --instance-type t2.micro especifica o tipo da instância, essa instância especificamente é muito usada para fins de estudo e testes.
O comando --key-name escola-da-nuvem-key associa o par de chaves SSH existente, criado no passo 1 à instância.
Este par de chaves é crucial para se conectar via SSH à instância após ela ser lançada.
O comando --security-group-ids $SG_ID associa a instância ao grupo de segurança SG_ID que foi criado no passo 6 e será o firewall virtual que irá controlar o tráfego de entrada e saída da instância, nesse caso permitindo todo o ingresso HTTP pela porta 80 e limitando o acesso SSH pela porta 22 somente ao IP público do seu equipamento que foi associado à variável MY_IP no passo 6.3.
O comando --subnet-id $SUBNET_ID associa a instância à sub-rede SUBNET_ID que foi criada no passo 3.
O comando --associate-public-ip-address instrui a AWS a atribuir um endereço IP público à sua instância.
Isso é fundamental para que a instância seja acessível diretamente da internet (por exemplo, para SSH ou para hospedar um site).
O comando --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=EscolaDaNuvem-WebApp}]' --query 'Instances[0].InstanceId' --output text) Indica que a tag será aplicada a uma instância, filtra a saída que seria em formato JSON para formato simples, a expressão ‘Instances[0].InstanceID’ seleciona o primeiro item da lista e extrai o valor da ID da instância, não mostra o conteúdo e salva o arquivo para ser usado nos próximos comandos na mesma sessão.
O comando echo "Aguardando a instância com ID: $INSTANCE_ID estar em execução..." imprime no terminal a frase “Aguardando a instância com ID: ” seguido pelo valor real que foi armazenado na variável INSTANCE_ID.
O comando aws ec2 wait instance-running --instance-ids $INSTANCE_ID é muito útil para automação e scripting, pois pausa a execução do script e fica monitorando até que a instância atinja o estado desejado.
O comando echo "Instância em execução!" imprime no terminal a frase “Instância em execução!”.

7.3-) OBTENHA O ENDEREÇO IP PÚBLICO DA INSTÂNCIA:
COMANDO:

Bash

PUBLIC_IP=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)
echo "O endereço IP público da sua instância é: $PUBLIC_IP"
OBSERVAÇÕES:
O comando PUBLIC_IP=$(...) executa tudo o que está dentro dos parênteses e atribui o resultado à variável chamada PUBLIC_IP.
O comando aws ec2 describe-instances --instance-ids $INSTANCE_ID busca as informações da instância com o ID armazenado na variável INSTANCE_ID.
O comando --query 'Reservations[0].Instances[0].PublicIpAddress' --output text filtra a saída que seria em formato JSON para formato simples, a expressão ‘Reservations[0].Instances[0].PublicIpAddress’ navega pela estrutura JSON para extrair o valor do endereço IP público da primeira instância encontrada na primeira reserva, não mostra o conteúdo e salva o arquivo para ser usado nos próximos comandos na mesma sessão.
O comando echo "O endereço IP público da sua instância é: $PUBLIC_IP" imprime no terminal a frase “O endereço IP público da sua instância é:” seguido pelo valor real do endereço IP que foi armazenado na variável PUBLIC_IP.

8-) INSTALANDO E CONFIGURANDO O SERVIDOR WEB NGINX:

8.1-) CONECTE À INSTÂNCIA VIA SSH:
COMANDO:

Bash

ssh -i "escola-da-nuvem-key.pem" ec2-user@$PUBLIC_IP

OBSERVAÇÕES:
O comando ssh é a ferramenta de linha de comando para conectar-se a servidores remotos.
O comando -i "escola-da-nuvem-key.pem" especifica o arquivo da chave privada que será usado para autenticação.
O comando ec2-user@$PUBLIC_IP é o nome de usuário (para AMIs Amazon Linux 2, o nome de usuário padrão é ec2-user) seguido pelo símbolo @ e o endereço IP público da instância.

8.2-) DENTRO DA INSTÂNCIA, ATUALIZE OS PACOTES E INSTALE O NGINX:
Bash

sudo yum update -y
sudo amazon-linux-extras install nginx1.12 -y

OBSERVAÇÕES:
O comando sudo yum update -y atualiza todos os pacotes instalados no sistema para suas versões mais recentes.
O comando sudo amazon-linux-extras install nginx1.12 -y instala o Nginx versão 1.12.

8.3-) INICIE O SERVIÇO DO NGINX E HABILITE-O PARA INICIAR COM O SISTEMA:
Bash

sudo systemctl start nginx
sudo systemctl enable nginx

OBSERVAÇÕES:
O comando sudo systemctl start nginx inicia o serviço Nginx, tornando o servidor web ativo.
O comando sudo systemctl enable nginx configura o serviço Nginx para iniciar automaticamente sempre que a instância for reiniciada.

8.4-) CRIE A PÁGINA WEB PERSONALIZADA:

8.4.1-) ACESSE O DIRETÓRIO RAIZ DO NGINX:
COMANDO:

Bash

cd /usr/share/nginx/html

OBSERVAÇÕES:
O comando cd /usr/share/nginx/html muda o diretório atual para a pasta onde o Nginx serve os arquivos web por padrão.

8.4.2-) CRIE OU EDITE O ARQUIVO index.html:

COMANDO:

Bash

sudo nano index.html

COLE O CÓDIGO HTML E CSS ABAIXO NO EDITOR NANO:

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

APÓS COLAR O CONTEÚDO NO nano:

Pressione Ctrl + X para sair.

Pressione Y para confirmar que deseja salvar as alterações.

Pressione Enter para confirmar o nome do arquivo (index.html).

8.4.3-) VERIFIQUE AS PERMISSÕES DO ARQUIVO:
COMANDO:

Bash

sudo chmod 644 index.html

OBSERVAÇÕES:
O comando sudo chmod 644 index.html garante que o Nginx tenha permissão para ler seu novo arquivo. As permissões 644 são ideais para arquivos web (leitura e escrita para o proprietário, apenas leitura para o grupo e outros).

8.4.4-) REINICIE O NGINX:
COMANDO:

Bash

sudo systemctl restart nginx

OBSERVAÇÕES:
O comando sudo systemctl restart nginx reinicia o Nginx para que comece a servir o novo conteúdo.

9-) TESTAR O SERVIDOR NGINX:
Abra um navegador e digite o endereço IP público da sua instância ($PUBLIC_IP). Você deverá ver a página personalizada que acabou de criar.

Observações Finais Importantes:
(¹): A AWS reserva os primeiros 4 endereços IP e o último endereço IP de cada sub-rede, totalizando 5 endereços que não podem ser utilizados para instâncias ou outros recursos dentro da sub-rede.

(²): Lembre-se que o seu endereço IP público pode mudar se você não tiver uma conexão com IP dedicado, nesse caso pode ser necessário que seja necessário rodar novamente o comando para descobrir o seu IP atual e armazenar na variável MY_IP.

Às vezes o editor de texto troca as aspas duplas por outro tipo de aspas ou o traço duplo por um travessão e isso causa erros, copie o erro e jogue no GEMINI ou ChatGPT e ele indicará qual foi o erro de digitação.

Limpeza da Infraestrutura
É extremamente importante deletar os recursos criados após concluir seus estudos para evitar cobranças indesejadas na sua conta AWS. Este script assume que você está na mesma sessão do Cloud Shell onde os comandos de criação foram executados, e as variáveis ($VPC_ID, $SUBNET_ID, etc.) ainda estão definidas.

LEIA ATENTAMENTE AS OBSERVAÇÕES DE CADA BLOCO DE COMANDO ANTES DE EXECUTAR O PRÓXIMO BLOCO DE COMANDOS NA MESMA SESSÃO DO CLOUD SHELL, PARA PODER UTILIZAR AS VARIÁVEIS CRIADAS.

COMANDO:

Bash

aws ec2 terminate-instances --instance-ids "$INSTANCE_ID"
aws ec2 wait instance-terminated --instance-ids "$INSTANCE_ID"
aws ec2 delete-security-group --group-id "$SG_ID"
aws ec2 delete-route --route-table-id "$ROUTE_TABLE_ID" --destination-cidr-block 0.0.0.0/0
aws ec2 delete-route-table --route-table-id "$ROUTE_TABLE_ID"
aws ec2 detach-internet-gateway --internet-gateway-id "$IGW_ID" --vpc-id "$VPC_ID"
aws ec2 delete-internet-gateway --internet-gateway-id "$IGW_ID"
aws ec2 delete-subnet --subnet-id "$SUBNET_ID"
aws ec2 delete-vpc --vpc-id "$VPC_ID"

OBSERVAÇÕES:
(¹): A AWS reserva os primeiros 4 endereços IP e o último endereço IP de cada sub-rede, totalizando 5 endereços que não podem ser utilizados para instâncias ou outros recursos dentro da sub-rede.

(²): Lembre-se que o seu endereço IP público pode mudar se você não tiver uma conexão com IP dedicado, nesse caso pode ser necessário que seja necessário rodar novamente o comando para descobrir o seu IP atual e armazenar na variável MY_IP.

Às vezes o editor de texto troca as aspas duplas por outro tipo de aspas ou o traço duplo por um travessão e isso causa erros, copie o erro e jogue no GEMINI ou ChatGPT e ele indicará qual foi o erro de digitação.


VERSION EN-US
Creating a Simple Nginx Server on AWS using Cloud Shell
DESCRIPTION: We will create a basic structure in AWS; in the end, we will have created our key pair, VPC, subnet, security group with SSH access releases on port 22 and HTTP access on port 80, an Internet Gateway for internet access, and an Nginx web server installed and displaying a personalized message. I believe it's interesting for everyone to change the names of the keys, networks, and put their own message and even images and links on the server's page if desired.

DO NOT USE THIS TUTORIAL FOR PRODUCTION, IT DOES NOT CONTAIN RECOMMENDED SECURITY PRACTICES AND THE INSTANCE AND SERVICES REMAIN EXTREMELY VULNERABLE!

Prerequisites
An active AWS account.

Access to AWS Cloud Shell (directly from the AWS console).

Step-by-Step
1-) CREATE KEY PAIR FOR SSH ACCESS:
COMMAND:

Bash

aws ec2 create-key-pair --key-name escola-da-nuvem-key --query 'KeyMaterial' --output text > escola-da-nuvem-key.pem
OBSERVATIONS:
The aws ec2 create-key-pair command creates the private key that will allow secure communication with the EC2 instance via the SSH protocol.
The --query 'KeyMaterial' --output text command filters the output, which would otherwise be in JSON format, into a simple text format, does not show the content, and saves the file.

1.1-) CHANGE FILE PERMISSIONS:
COMMAND:

Bash

chmod 400 escola-da-nuvem-key.pem
1.2-) DOWNLOAD THE PRIVATE KEY TO YOUR COMPUTER:
In the top right corner of Cloud Shell, click ACTIONS.

Select Download File.

In the window that opens, type the path to your key:
/home/cloudshell-user/escola-da-nuvem-key.pem

Click Download.

2-) CREATE THE VPC (Virtual Private Cloud):
COMMAND:

Bash

VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text)
OBSERVATIONS:
The VPC_ID=$(...) command executes everything within the parentheses and assigns the result to the variable named VPC_ID.
The aws ec2 create-vpc command creates a new private and isolated virtual network in the AWS cloud for your project.
The --cidr-block 10.0.0.0/16 command defines the block of private addresses that your network will use; the /16 mask provides 65,536¹ possible IP addresses. The definition of the network size within the VPC must be carefully studied, as if it is too large, it may have overlapping issues when communicating with another VPC, and if it is too small, it may run out of addresses for your instances.
The --query Vpc.VpcID –output text command filters the output, which would otherwise be in JSON format, into a simple text format, does not show the content, and saves the file to be used in subsequent commands in the same session.

2.1-) ADD A NAME TO THE VPC FOR EASY IDENTIFICATION:
COMMAND:

Bash

aws ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=EscolaDaNuvemVPC
OBSERVATIONS:
The aws ec2 create-tags command assigns metadata (tags) to the chosen resource in EC2.
The --resources $VPC_ID command determines which resource will receive the metadata.
The --tags Key=Name,Value=EscolaDaNuvemVPC command defines the tag.

2.2-) LIST INFORMATION ABOUT YOUR VPC:
COMMAND:

Bash

aws ec2 describe-vpcs --filters "Name=tag:Name,Values=EscolaDaNuvemVPC"
OBSERVATIONS:
The aws ec2 --describe-vpcs command describes all your VPCs.
The --filters “Name=tag:Name,Values=Escola DaNuvemVPC” command filters by the name (tag) that was assigned in the previous command.

3-) CREATE SUBNET:
COMMAND:

Bash

SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 --query Subnet.SubnetId --output text)
OBSERVATIONS:
The SUBNET_ID=$(...) command executes everything within the parentheses and assigns the result to the variable named SUBNET_ID.
The aws ec2 --create subnet command creates a new subnet.
The --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 command creates the subnet within your VPC and defines the block of private addresses that your subnet will use; the /24 mask provides 256¹ possible IP addresses. The definition of the subnet size within the VPC must be carefully studied, as if it is too large, it may have overlapping issues when communicating with other subnets within your VPC, and if it is too small, it may run out of addresses for your instances.
The --query Subnet.SubnetID --output text command filters the output, which would otherwise be in JSON format, into a simple text format, does not show the content, and saves the file to be used in subsequent commands in the same session.

3.1-) ADD A NAME TO THE SUBNET FOR EASY IDENTIFICATION:
COMMAND:

Bash

aws ec2 create-tags --resources $SUBNET_ID --tags Key=Name,Value=EscolaDaNuvemSubnet
echo “Subnet Created with ID: $SUBNET_ID”
OBSERVATIONS:
The aws ec2 create-tags command assigns metadata (tags) to the chosen resource in EC2.
The --resources $SUBNET_ID command determines which resource will receive the metadata.
The --tags Key=Name,Value=EscolaDaNuvemSubnet command defines the tag.
The echo “Subnet Created with ID: $SUBNET_ID” command prints the phrase "Subnet Created with ID: " followed by the actual ID value of the subnet that was stored in the SUBNET_ID variable to the terminal.

4-) CREATE AN INTERNET GATEWAY:
COMMAND:

Bash

IGW_ID=$(aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text)
OBSERVATIONS:
The IGW_ID=$(...) command executes everything within the parentheses and assigns the result to the variable named IGW_ID.
The aws ec2 create-internet-gateway command creates an Internet Gateway so that your web server can be accessed via the internet.
The --query InternetGateway.InternetGatewayID --output text command filters the output, which would otherwise be in JSON format, into a simple text format, does not show the content, and saves the file to be used in subsequent commands in the same session.

4.1-) ADD A NAME TO THE INTERNET GATEWAY FOR EASY IDENTIFICATION:
COMMAND:

Bash

aws ec2 create-tags --resources $IGW_ID --tags Key=Name,Value=EscolaDaNuvemIGW
OBSERVATIONS:
The aws ec2 create-tags command assigns metadata (tags) to the chosen resource in EC2.
The --resources $IGW_ID command determines which resource will receive the metadata.
The --tags Key=Name,Value=EscolaDaNuvemSubnet command defines the tag.

4.2-) ATTACH THE INTERNET GATEWAY TO THE VPC:
COMMAND:

Bash

aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID
OBSERVATIONS:
The aws ec2 attach-internet-gateway command attaches the internet gateway to your VPC.
The --vpc-id $VPC_ID command specifies which VPC it is.
The --internet-gateway-id $IGW_ID command specifies which Internet Gateway it is.

COMMAND:

Bash

echo “Internet Gateway Created with ID: $IGW_ID and attached to VPC”
OBSERVATIONS:
The echo “Internet Gateway Created with ID: $IGW_ID and attached to VPC” command prints the phrase "Internet Gateway Created with ID: " followed by the actual ID value of the Internet Gateway that was stored in the IGW_ID variable to the terminal.

5-) CREATE A ROUTE TABLE:
COMMAND:

Bash

ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query RouteTableId --output text)
OBSERVATIONS:
The ROUTE TABLE ID=$(...) command executes everything within the parentheses and assigns the result to the variable named ROUTE_TABLE_ID.
The aws ec2 create-route-table --vpc=id $VPC_ID command creates a route table in the VPC whose ID was stored in the VPC_ID variable.
The --query RouteTableID --output text command filters the output, which would otherwise be in JSON format, into a simple text format, does not show the content, and saves the file to be used in subsequent commands in the same session.

5.1-) ADD A NAME TO THE ROUTE TABLE FOR EASY IDENTIFICATION:
COMMAND:

Bash

aws ec2 create-tags --resources $ROUTE_TABLE_ID --tags Key=Name,Value=EscolaDaNuvemRouteTable
OBSERVATIONS:
The aws ec2 create-tags command assigns metadata (tags) to the chosen resource in EC2.
The --resources $ROUTE_TABLE_ID command determines which resource will receive the metadata.
The --tags Key=Name,Value=EscolaDaNuvemRouteTable command defines the tag.

5.2-) CREATE A ROUTE TO THE INTERNET:
COMMAND:

Bash

aws ec2 create-route --route-table-id $ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
OBSERVATIONS:
The aws ec2 create-route command creates a route within the route table with the ID stored in the ROUTE_TABLE_ID variable.
The --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID command is instructing the route table to send all traffic that is not for the VPC itself (or for other specific routes) to the Internet Gateway specified by the IGW_ID variable.

5.3-) ASSOCIATE THE ROUTE TABLE WITH THE SUBNET:
COMMAND:

Bash

aws ec2 associate-route-table --subnet-id $SUBNET_ID --route-table-id $ROUTE_TABLE_ID
OBSERVATIONS:
The aws ec2 associate-route-table command associates a route with a subnet.
The --subnet-id $SUBNET_ID --route-table-id $ROUTE_TABLE_ID command specifies which subnet will be associated with a given route.

6-) CREATE A SECURITY GROUP:
COMMAND:

Bash

SG_ID=$(aws ec2 create-security-group --group-name EscolaDaNuvemSG --description "SSH and HTTP Access" --vpc-id $VPC_ID --query GroupId --output text)
OBSERVATIONS:
The SG_ID=$(...) command executes everything within the parentheses and assigns the result to the variable named SG_ID.
The aws ec2 create-security-group command creates the security group.
The --group-name EscolaDaNuvemSG command defines the name of your security group.
The --description “SSH and HTTP Access” command describes the security group.
The --vpc-id $VPC_ID command specifies that the security group will be created in the VPC whose variable is VPC_ID.
The --query GroupId --output text command filters the output, which would otherwise be in JSON format, into a simple text format, does not show the content, and saves the file to be used in subsequent commands in the same session.

6.2-) ADD A NAME TO THE ROUTE TABLE FOR EASY IDENTIFICATION:
COMMAND:

Bash

aws ec2 create-tags --resources $SG_ID --tags Key=Name,Value=EscolaDaNuvemSG
echo "Security Group Created with ID: $SG_ID”
OBSERVATIONS:
The aws ec2 create-tags command assigns metadata (tags) to the chosen resource in EC2.
The --resources $SG_ID command determines which resource will receive the metadata.
The --tags Key=Name,Value=EscolaDaNuvemSG command defines the tag.
The echo "Security Group Created with ID: $SG_ID command prints the phrase "Security Group Created with ID:” followed by the actual ID value of the Internet Gateway that was stored in the SG_ID variable to the terminal.

6.3-) ALLOW SSH ACCESS (port 22) AND HTTP (port 80):
COMMAND:

Bash

MY_IP=$(curl -s http://checkip.amazonaws.com)/32
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 22 --cidr $MY_IP
OBSERVATIONS:
The MY_IP=$(curl -s http://checkip.amazonaws.com)/32 command uses the curl tool and the AWS service to discover your current public IP and store it in the MY_IP variable.
The aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 22 --cidr $MY_IP command adds an ingress rule to the security group whose variable is SG_ID, allowing SSH traffic (port 22) to reach it from your current IP².

COMMAND:

Bash

aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0
OBSERVATIONS:
The aws ec2 authorize-security-group-ingress --group-id $SG_ID command adds an ingress rule to the security group whose variable is SG_ID.
The --protocol tcp --port 80 --cidr 0.0.0.0/0 command allows TCP traffic (port 80) to reach it from any IPV4 address.

7-) LAUNCH THE AMAZON LINUX EC2 INSTANCE:
7.1-) GET THE AMI ID:
COMMAND:

Bash

AMI_ID=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].Value' --output text)
echo "Using AMI ID: $AMI_ID"
OBSERVATIONS:
The AMI_ID=$(...) command executes everything within the parentheses and assigns the result to the variable named AMI_ID.
The aws ssm get-parameters command instructs the SSM (AWS System Manager) to extract a value from the Parameter Store.
The --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 command specifies the name (path) of the parameter you want to get from the Parameter Store, which always contains the latest Amazon Linux 2 AMI ID.
The --query 'Parameters[0].Value' --output text command filters the output, which would otherwise be in JSON format, into a simple text format. The expression ‘Parameters[0].Value’ selects the first item in the list and extracts the AMI ID value, does not show the content, and saves the file to be used in subsequent commands in the same session.
The echo "Using AMI ID: $AMI_ID" command prints the phrase "Using AMI ID:” followed by the actual AMI ID value that was stored in the AMI_ID variable to the terminal.

7.2-) CREATE THE EC2 INSTANCE:
COMMAND:

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
WAIT A FEW MOMENTS AND THEN ENTER THE NEXT BLOCK:

COMMAND:

Bash

echo "Waiting for instance with ID: $INSTANCE_ID to be running..."
aws ec2 wait instance-running --instance-ids $INSTANCE_ID
echo "Instance running!"
OBSERVATIONS:
In this sequence of concatenated commands, all variables created in the previous commands are used. This is a common and very useful practice for automation, as cloud infrastructure is rarely static.
Scenarios such as multiple environments (development, testing, production), different geographical regions, or changing requirements are common, as well as for code reuse, consistency, and ease of maintenance.
The INSTANCE_ID=$(...) command executes everything within the parentheses and assigns the result to the variable named INSTANCE_ID.
The aws ec2 run-instances command launches one or more (can be used to launch in batches) new instances.
The --image-id $AMI_ID command specifies which AMI (Amazon Machine Image) will be used to create the instance, and the AMI_ID variable is the one that was created in step 7.1.
The --instance-type t2.micro command specifies the instance type; this specific instance is widely used for study and testing purposes.
The --key-name escola-da-nuvem-key command associates the existing SSH key pair, created in step 1, with the instance.
This key pair is crucial for connecting via SSH to the instance after it is launched.
The --security-group-ids $SG_ID command associates the instance with the SG_ID security group that was created in step 6, and it will be the virtual firewall that will control inbound and outbound traffic for the instance. In this case, it allows all HTTP ingress on port 80 and limits SSH access on port 22 only to your device's public IP that was associated with the MY_IP variable in step 6.3.
The --subnet-id $SUBNET_ID command associates the instance with the SUBNET_ID subnet that was created in step 3.
The --associate-public-ip-address command instructs AWS to assign a public IP address to your instance.
This is fundamental for the instance to be directly accessible from the internet (e.g., for SSH or to host a website).
The --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=EscolaDaNuvem-WebApp}]' --query 'Instances[0].InstanceId' --output text) command indicates that the tag will be applied to an instance, filters the output which would otherwise be in JSON format to a simple text format. The expression ‘Instances[0].InstanceID’ selects the first item in the list and extracts the instance ID value, does not show the content, and saves the file to be used in subsequent commands in the same session.
The echo "Waiting for instance with ID: $INSTANCE_ID to be running..." command prints the phrase "Waiting for instance with ID: " followed by the actual value stored in the INSTANCE_ID variable to the terminal.
The aws ec2 wait instance-running --instance-ids $INSTANCE_ID command is very useful for automation and scripting, as it pauses script execution and monitors until the instance reaches the desired state.
The echo "Instance running!" command prints the phrase "Instance running!" to the terminal.

7.3-) GET THE INSTANCE'S PUBLIC IP ADDRESS:
COMMAND:

Bash

PUBLIC_IP=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)
echo "Your instance's public IP address is: $PUBLIC_IP"
OBSERVATIONS:
The PUBLIC_IP=$(...) command executes everything within the parentheses and assigns the result to the variable named PUBLIC_IP.
The aws ec2 describe-instances --instance-ids $INSTANCE_ID command retrieves information about the instance with the ID stored in the INSTANCE_ID variable.
The --query 'Reservations[0].Instances[0].PublicIpAddress' --output text command filters the output, which would otherwise be in JSON format, into a simple text format. The expression ‘Reservations[0].Instances[0].PublicIpAddress’ navigates through the JSON structure to extract the public IP address value of the first instance found in the first reservation, does not show the content, and saves the file to be used in subsequent commands in the same session.
The echo "Your instance's public IP address is: $PUBLIC_IP" command prints the phrase "Your instance's public IP address is:" followed by the actual IP address value that was stored in the PUBLIC_IP variable to the terminal.

8-) INSTALLING AND CONFIGURING THE NGINX WEB SERVER:
8.1-) CONNECT TO THE INSTANCE VIA SSH:
COMMAND:

Bash

ssh -i "escola-da-nuvem-key.pem" ec2-user@$PUBLIC_IP
OBSERVATIONS:
The ssh command is the command-line tool for connecting to remote servers.
The -i "escola-da-nuvem-key.pem" command specifies the private key file to be used for authentication.
The ec2-user@$PUBLIC_IP command is the username (for Amazon Linux 2 AMIs, the default username is ec2-user) followed by the @ symbol and the instance's public IP address.

8.2-) INSIDE THE INSTANCE, UPDATE PACKAGES AND INSTALL NGINX:
Bash

sudo yum update -y
sudo amazon-linux-extras install nginx1.12 -y
OBSERVATIONS:
The sudo yum update -y command updates all installed packages on the system to their latest versions.
The sudo amazon-linux-extras install nginx1.12 -y command installs Nginx version 1.12.

8.3-) START THE NGINX SERVICE AND ENABLE IT TO START WITH THE SYSTEM:
Bash

sudo systemctl start nginx
sudo systemctl enable nginx
OBSERVATIONS:
The sudo systemctl start nginx command starts the Nginx service, making the web server active.
The sudo systemctl enable nginx command configures the Nginx service to start automatically whenever the instance is restarted.

8.4-) CREATE THE CUSTOM WEB PAGE:
8.4.1-) ACCESS THE NGINX ROOT DIRECTORY:
COMMAND:

Bash

cd /usr/share/nginx/html
OBSERVATIONS:
The cd /usr/share/nginx/html command changes the current directory to the folder where Nginx serves web files by default.

8.4.2-) CREATE OR EDIT THE index.html FILE:
COMMAND:

Bash

sudo nano index.html
PASTE THE HTML AND CSS CODE BELOW INTO THE NANO EDITOR:

HTML

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>USEFUL LINKS REPOSITORY AWS DEVELOPER ASSOCIATE COURSE CLOUD SCHOOL</title>
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
<style>
body {
font-family: 'Roboto', sans-serif; /* Modern font */
background-color: #ADD8E6; /* Light blue */
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
<h1>USEFUL LINKS REPOSITORY AWS DEVELOPER ASSOCIATE COURSE CLOUD SCHOOL</h1>
<ul>
<li><a href="https://aws.amazon.com/certification/certified-developer-associate/" target="_blank">AWS Certified Developer - Associate Certification (Official AWS Page)</a></li>
<li><a href="https://www.examprepper.co/exam/25/1" target="_blank">Exam Prepper - AWS Developer Associate Practice Exams</a></li>
<li><a href="https://www.youtube.com/playlist?list=PLK2b5y9F1DqZRzd5x69Z4ndR7jsPj7ghr" target="_blank">Youtube Content via Google User Content</a></li>
</ul>
<p class="footer">Created for the AWS Developer Associate Course</p>
</div>
</body>
</html>
AFTER PASTING THE CONTENT INTO nano:

Press Ctrl + X to exit.

Press Y to confirm that you want to save the changes.

Press Enter to confirm the file name (index.html).

8.4.3-) CHECK FILE PERMISSIONS:
COMMAND:

Bash

sudo chmod 644 index.html
OBSERVATIONS:
The sudo chmod 644 index.html command ensures that Nginx has permission to read your new file. Permissions 644 are ideal for web files (read and write for the owner, read-only for the group and others).

8.4.4-) RESTART NGINX:
COMMAND:

Bash

sudo systemctl restart nginx
OBSERVATIONS:
The sudo systemctl restart nginx command restarts Nginx so that it begins serving the new content.

9-) TEST THE NGINX SERVER:
Open a browser and type your instance's public IP address ($PUBLIC_IP). You should see the custom page you just created.

Important Final Notes:
(¹): AWS reserves the first 4 IP addresses and the last IP address of each subnet, totaling 5 addresses that cannot be used for instances or other resources within the subnet.

(²): Remember that your public IP address may change if you do not have a dedicated IP connection. In this case, it may be necessary to run the command again to discover your current IP and store it in the MY_IP variable.

Sometimes the text editor swaps double quotes for another type of quote or a double dash for an em dash, and this causes errors. Copy the error and paste it into GEMINI or ChatGPT, and it will indicate what the typing error was.

Infrastructure Cleanup
It is extremely important to delete the created resources after completing your studies to avoid unwanted charges on your AWS account. This script assumes that you are in the same Cloud Shell session where the creation commands were executed, and the variables ($VPC_ID, $SUBNET_ID, etc.) are still defined.

READ THE OBSERVATIONS OF EACH COMMAND BLOCK CAREFULLY BEFORE EXECUTING THE NEXT BLOCK OF COMMANDS IN THE SAME CLOUD SHELL SESSION, TO BE ABLE TO USE THE CREATED VARIABLES.

COMMAND:

Bash

aws ec2 terminate-instances --instance-ids "$INSTANCE_ID"
aws ec2 wait instance-terminated --instance-ids "$INSTANCE_ID"
aws ec2 delete-security-group --group-id "$SG_ID"
aws ec2 delete-route --route-table-id "$ROUTE_TABLE_ID" --destination-cidr-block 0.0.0.0/0
aws ec2 delete-route-table --route-table-id "$ROUTE_TABLE_ID"
aws ec2 detach-internet-gateway --internet-gateway-id "$IGW_ID" --vpc-id "$VPC_ID"
aws ec2 delete-internet-gateway --internet-gateway-id "$IGW_ID"
aws ec2 delete-subnet --subnet-id "$SUBNET_ID"
aws ec2 delete-vpc --vpc-id "$VPC_ID"
OBSERVATIONS:
(¹): AWS reserves the first 4 IP addresses and the last IP address of each subnet, totaling 5 addresses that cannot be used for instances or other resources within the subnet.

(²): Remember that your public IP address may change if you do not have a dedicated IP connection. In this case, it may be necessary to run the command again to discover your current IP and store it in the MY_IP variable.

Sometimes the text editor swaps double quotes for another type of quote or a double dash for an em dash, and this causes errors. Copy the error and paste it into GEMINI or ChatGPT, and it will indicate what the typing error was.

