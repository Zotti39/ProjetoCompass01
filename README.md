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
