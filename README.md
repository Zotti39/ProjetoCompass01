# Primeiro projeto do PB Compass
Este projeto faz parte do programa de bolsas DevSecOps da compass, iniciado em dezembro/2023 e tem como objetivo implementar na pratica os conhecimentos adquiridos nesse primeiro mes e meio de estagio, tempo usado para fazer alguns cursos sobre administracao de sistemas linux e uso da Amazon AWS para criacao de recursos de computacao em nuvem.

A seguir sera documentada a criacao de um ambiente com os recursos da AWS e a instalacao de um sistema linux nesse ambiente

# Arquitetura do ambiente AWS
Os seguintes itens foram usados/criados para o ambiente do projeto:
- 1 VPC (Projeto01-vpc)
  - 2 Subnets publicas nas AZs us-east-1a e us-east-1b
  - 1 Internet Gateway (Projeto01-igw)
  - 1 Route Table ligando ambas as subnets publicas a internet gateway
- 1 Security Group (SGProjeto01) com as portas 22/TCP, 80/TCP, 443/TCP, 2049/TCP, 2049/UDP, 111/TCP, 111/UDP liberadas para trafego de entrada vindo de qualquer origem, que sera utilizado como firewall da instancia ec2.
- Instancia EC2 (EC2Projeto01) do tipo t3.small, com 1 volume de 16 GiB e um Elastic IP Address associado (52.70.182.61), utiliza a chave ChaveProjeto01.pem disponivel neste repositorio. 

# Ambiente Linux
Esta documenta;ao leva em considera;ao que estejamos utilizando uma maquina com sistema operacional linux, mas caso voce esteja utilizando uma maquina com windows, pode utilizar o PuTTY, e precisara instala-lo e converter a chave de .pem para .ppk 

A primeira configuracao realizada no terminal da instancia foi a modificacao das permissoes da chave SSH atribuida a instancia, para isso foi usada o seguinte comando:

`chmod 400 ChaveProjeto01.pem`

Apos feito isso podemos nos conectar a instancia EC2 usando SSH com a chave privada, usando o seguinte comando:

` ssh -i ChaveProjeto01.pem ec2-user@52.70.182.61 `

Para garantir que o sistema esteja atualizado passamos um comando update e upgrade na maquina:

`sudo yum update ; sudo yum upgrade`

### Instala;'ao do NFS

Com o sistema atualizado e funcionando corretamente, podemos instalar o pacote NFS, para isso usei o seguinte comando:

`sudo yum -y install nfs-utils`

E em seguida criaremos um diretorio com o nome "Gabriel" dentro do FileSystem do NFS, e defiinremos as permissoes deste diretorio, utilizando os seguintes comandos:

`sudo mkdir /mnt/Gabriel`

`cd /Gabriel`

`sudo chmod go+rw .`

E para inicializar e permitir o NFS usaremos:

`sudo systemctl start nfs-server`

`sudo systemctl enable nfs-server`

Agora temos o NFS instalado e ativado dentro da nossa instancia EC2.
