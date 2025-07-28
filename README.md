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

