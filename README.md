# FULL-SUBVERSION-SVN :turtle: 
## FULL SUBVERSION PROJECT (SVN, TRAC, BACKUP FULL + INCREMENTAL and RESTORY BACKUP)

## 1. DESCRIÇÃO DO DOCUMENTO
O objetivo desta documentação é especificar as atividades necessárias para a instalação, configuração e implantação do subversion SVN e TRAC em um ambiente Linux. Para construção deste projeto, foi usado como distribuição o Debian 10 buster.

Scripts para automatizar alguns processos também estão disponibilizados neste projeto, como: criação de usuários, criação de projetos, Backup full + incremental e restauração de Backup.


## 2. INSTALAÇÃO DO SISTEMA
Recomenda-se a instalação da distribuição no formatado LVM. Definindo o Volume lógico pelo menos a estrutura de diretório "/var", pois será nessa estrutura onde alocaremos os projetos.


### 2.1 Preparação do sistema
Inclusão das linhas no arquivo /etc/apt/sources.list

**~# nano /etc/apt/sources.list**

```
deb http://debian.c3sl.ufpr.br/debian/ buster main non-free contrib
deb-src http://debian.c3sl.ufpr.br/debian/ buster main non-free contrib
deb http://security.debian.org/debian-security buster/updates main contrib non-free
deb-src http://security.debian.org/debian-security buster/updates main contrib non-free
deb http://debian.c3sl.ufpr.br/debian/ buster-updates main contrib non-free
deb-src http://debian.c3sl.ufpr.br/debian/ buster-updates main contrib non-free
```

### 2.2 Atualizar o sistema

**~# apt update -y && apt upgrade -y**

`apt update` atualiza a lista de pacotes e programas que podem ser instalados na máquina | `apt upgrade` atualiza o sistema e baixa e instala atualizações de pacotes e dos programas da máquina.


### 2.3 Instalação de pacotes necessários

**~# apt-get install vim**

>_"instala o editor de texto vi"_

**~# apt-get install dialog**

>_“permite criação de projeto e usuário de forma interativa”_


### 2.4 Configurando o VIM ###

Altere o conteúdo do arquivo .bashrc que fica localizado ocultamente no diretório /root com o seguinte conteúdo [`.bashrc`](https://github.com/clayton-camargo/FULL-SUBVERSION-SVN/blob/main/bashrc) e por fim descomentar a linha `sintax on` no diretório /etc/vim/vimrc.


### 2.5 Instalando pacotes necessários para SVN e TRAC ###

**~# apt-get install subversion && apt-get install subversion-tools**

**~# apt-get install apache2 && apt-get install libapache2-mod-svn**

**~# apt-get install apache2-utils && apt-get install libapache2-mod-python**

**~# apt-get install trac**


## 3.	CONFIGURAÇÃO DO APACHE
Edite o arquivo [`default.conf`](https://github.com/clayton-camargo/FULL-SUBVERSION-SVN/blob/main/default.conf) com os seguintes dados:

**~# vi /etc/apache2/sites-available/default.conf**
```
<VirtualHost *:80>
        ServerAdmin youremail@yourdomain.com #Alterar para o seu#
        ServerName server.com.br
        ServerAlias www.server.com.br
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <Directory "/var/www/html">
                Options FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
        </Directory>

</VirtualHost>
```


## 4. CONFIGURAÇÃO DO SVN
Crie um arquivo [`svn.conf`](https://github.com/clayton-camargo/FULL-SUBVERSION-SVN/blob/main/svn.conf) dentro de /etc/apache/sites-available:

**~# vi /etc/apache2/sites-available/svn.conf**
>_Insira o conteúdo dentro de svn.conf:_
```
<Location /svn>
        DAV svn
        SVNParentPath /var/lib/svn
        AuthzSVNAccessFile /usr/local/etc/svn_authz.conf
        Satisfy Any
        Require valid-user
        AuthType Basic
        AuthName "Repositorios SVN"
        AuthUserFile /etc/apache2/dav_svn.passwd
</Location>

<Directory /var/lib/svn>
        AllowOverride None
        Order Deny,Allow
        Allow from all
</Directory>
```
