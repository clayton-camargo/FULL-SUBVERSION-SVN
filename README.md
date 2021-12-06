# FULL-SUBVERSION-SVN
# FULL SUBVERSION PROJECT (SVN, TRAC, BACKUP FULL + INCREMENTAL and RESTORY BACKUP)

# 1. DESCRIÇÃO DO DOCUMENTO
O objetivo desta documentação é especificar as atividades necessárias para a instalação, configuração e implantação do subversion SVN e TRAC em um ambiente Linux. Para construção deste projeto, foi usado como distribuição o Debian 10 buster.
Scripts para automatizar alguns processos também estão disponibilizados neste projeto, como: criação de usuários, criação de projetos, Backup full + incremental e restauração de Backup.

# 2. INSTALAÇÃO DO SISTEMA
Recomenda-se a instalação da distribuição no formatado LVM. Definindo o Volume lógico pelo menos a estrutura de diretório "/var", pois será nessa estrutura onde alocaremos os projetos.

2.1 **Preparação do sistema
Inclusão das linhas no arquivo /etc/apt/sources.list
~# nano /etc/apt/sources.list