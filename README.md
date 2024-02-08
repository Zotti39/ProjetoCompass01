# Primeiro projeto do PB Compass


 1. [Arquitetura do ambiente AWS](#AWS)
 2. [Ambiente Linux](#linux)
    -  [Instalando o NFS](#NFS)
    -  [Instalando o Apache](#apache)
    -  [Criação do script](#script)
    -  [Automatização do script](#automatizacao)

_Este Documento foi atualizado pela ultima vez em 08/02/2024_

Este projeto faz parte do programa de bolsas DevSecOps da compass, iniciado em dezembro/2023 e tem como objetivo implementar na prática os conhecimentos adquiridos nesse primeiro mês  e meio de estágio, tempo usado para fazer alguns cursos sobre administração de sistemas linux e uso da Amazon AWS para criação de recursos de computação em nuvem

A seguir será documentada a criação de um ambiente com os recursos da AWS e a instalação de um sistema linux com NFS, Apache, um script que valide se o serviço do apache está online e envie o resultado da validação para o diretorio no nfs
<div id='AWS'/>   

#
 
# 1.Arquitetura do ambiente AWS
Os seguintes itens foram usados/criados para o ambiente do projeto:
- Chave pública  (ChaveProjeto01)
- VPC (Projeto01-vpc)
  - 2 Subnets públicas nas AZs us-east-1a e us-east-1b
  - 1 Internet Gateway (Projeto01-igw)
  - 1 Route Table ligando ambas as subnets públicas ao internet gateway
<img src="https://github.com/Zotti39/ProjetoCompass01/blob/main/Screenshots/Captura%20de%20Tela%20(2).png">

- Security Group (SGProjeto01) com as portas 22/TCP, 80/TCP, 443/TCP, 2049/TCP, 2049/UDP, 111/TCP, 111/UDP liberadas para tráfego de entrada vindo de qualquer origem, que será utilizado como firewall da instância ec2.

<img src="https://github.com/Zotti39/ProjetoCompass01/blob/main/Screenshots/Captura%20de%20Tela%20(20).png">
  
- Instância EC2 (EC2Projeto01) do tipo t3.small, com 1 volume de 16 GiB e um Elastic IP Address associado (52.70.182.61), utiliza a chave `ChaveProjeto01.pem`, disponivel neste repositorio.

<div id='linux'/> 



 
# 2.Ambiente Linux

Esta documentação leva em consideração que estamos utilizando uma máquina com sistema operacional linux, mas caso voce esteja utilizando windows, o PuTTY pode ser usado como alternativa, e precisará ser instalado e converter a chave de acesso de .pem para .ppk 

A primeira configuração realizada no terminal da instância foi a modificação das permissões da chave SSH atribuída a instância, para isso foi usada o seguinte comando:

    chmod 400 ChaveProjeto01.pem 

Após feito isso podemos nos conectar a instância EC2 usando SSH com a chave privada, usando o seguinte comando:

    ssh -i ChaveProjeto01.pem ec2-user@52.70.182.61 

Para garantir que o sistema esteja atualizado passamos um comando update e upgrade no terminal da instância:

    sudo yum update ; sudo yum upgrade 





<div id='NFS'/> 
 
## - Instalando o NFS

Com o sistema atualizado e funcionando corretamente, podemos instalar o pacote NFS, para isso usamos o seguinte comando:

    sudo yum -y install nfs-utils 

E em seguida criaremos um diretório com o nome "Gabriel" dentro do FileSystem do NFS, e definimos as permissoes deste diretório, utilizando os seguintes comandos:

    sudo mkdir /mnt/Gabriel
######
    cd /mnt/Gabriel 
######
    sudo chmod go+rw . 

E para inicializar e permitir o NFS usaremos:

    sudo systemctl start nfs-server ; sudo systemctl enable nfs-server 

Agora temos o NFS instalado e ativado dentro da nossa instancia EC2.





<div id='apache'/> 
 
## - Instalando o Apache

Para instalar o apache passamos os seguintes comandos:

    sudo yum install httpd 

Como a instalação veio sem o arquivo da página inicial do apache, criaremos um html simples dentro do diretorio `/var/www/html` com o nome de `index.html` e daremos as permissões necessarias para cumprir com esta função:

    vi /var/www/html/index.html 
######
    sudo chmod 777 /var/www/html/index.html 

<img src="https://github.com/Zotti39/ProjetoCompass01/blob/main/Screenshots/Captura%20de%20Tela%20(4).png">

E após isso podemos iniciar e habilitar o início automático do apache com os comandos:

    sudo systemctl start httpd ; sudo systemctl enable httpd 
<div id='script'/> 
 
## - Criação do script

Tendo como objetivo fazer um histórico de logs do status de atividade do servidor apache, criamos um script chamado `scriptApache.sh` dentro do diretório `/home/ec2-user` que registra os resultados da verificação no formato `data -> ativo/inativo` e na ultima linha dentro do seu respectivo arquivo, `apache_logs_ativo.txt` ou `apache_logs_inativo.txt` 

Neste script usamos a linguagem bash, segue seu conteudo abaixo:

<img src="https://github.com/Zotti39/ProjetoCompass01/blob/main/Screenshots/Captura%20de%20Tela%20(8).png">

Com o script concluído, é necessário tambem dar permissões de execução ao arquivo .sh:

    sudo chmod +x scriptApache.sh 

<img src="https://github.com/Zotti39/ProjetoCompass01/blob/main/Screenshots/Captura%20de%20Tela%20(5).png">

Antes de configurar sua realização automática de 5 em 5 minutos, realizei um teste para ver se o script estava criando e armazenando os logs dentro dos arquivos corretamente, utilizando o seguinte comando:

    bash scriptApache.sh 

<img src="https://github.com/Zotti39/ProjetoCompass01/blob/main/Screenshots/Captura%20de%20Tela%20(10).png">

Para garantir que a parte que registra o log inativo tambem está funcionando, eu desliguei manualmente o apache e testei novamente o script, obtendo o seguinte resultado:

<img src="https://github.com/Zotti39/ProjetoCompass01/blob/main/Screenshots/Captura%20de%20Tela%20(11).png">
<div id='automatizacao'/> 
 
## - Automatização do script

Agora, sabendo que o script está funcionando corretamente, podemos passar os comandos que automatizam a realização do script a cada 5 minutos, para isso usaremos um Cron Job, que permite o agendamento de tarefas em servidores Unix/Linux

Como o crontab não está disponível na máquina ec2, teremos que instala-lo manualmente, usando o seguinte comando:

    sudo yum install cronie 

<img src="https://github.com/Zotti39/ProjetoCompass01/blob/main/Screenshots/Captura%20de%20Tela%20(13).png">

Tambem é necessario ativar e validar a inicialização automática do cron, para isso usamos os seguintes comandos:

    sudo systemctl start crond ; sudo systemctl enable crond

Agora podemos configurar a automação do script

    crontab -e 

E dentro do arquivo inserimos a seguinte linha de comando que automatiza a execução do script 

    */5 * * * * bash /home/ec2-user/scriptApache.sh 

Apos isso, para salvar o arquivo `Esc` -> `:wq` + `Enter` 

<img src="https://github.com/Zotti39/ProjetoCompass01/blob/main/Screenshots/Captura%20de%20Tela%20(18).png">

Acima vemos o resultado do teste do script, que a cada 5 minutos verificamos o arquivo de logs `apache_logs_ativo.txt` e vemos que uma nova linha foi adicionada a ele informando a hora da verificação e que o servidor estava ativo, assim concluindo com êxito o objetivo da atividade.
