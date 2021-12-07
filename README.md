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
### 4.1. Ativar o host virtual de subversão

**~# a2ensite svn.conf**

## 5. CONFIGURAÇÃO DO TRAC
Trac é um sistema de rastreamento de problemas e wiki aprimorado para projetos de desenvolvimento de software. A Trac usa uma abordagem minimalista para gerenciamento de projetos de software baseado na Web.

Ele fornece uma interface para Subversão e Git, uma Wiki integrada e instalações de relatórios convenientes.

Crie um arquivo [`trac.conf`](https://github.com/clayton-camargo/FULL-SUBVERSION-SVN/blob/main/trac.conf) dentro de /etc/apache/sites-available:

**~# vi /etc/apache2/sites-available/trac.conf**

>_Insira o conteúdo dentro de trac.conf:_
```
<Location /projetos>
        SetHandler mod_python
        PythonHandler trac.web.modpython_frontend
        PythonOption TracEnvParentDir /var/lib/trac/
        PythonOption TracUriRoot /projetos
        PythonPath "sys.path + ['/usr/local/lib/python2.7/site-packages/trac']"
        SetEnv PYTHON_EGG_CACHE /usr/local/lib/python2.7/site-packages/
        #PythonDebug On # apenas em caso de problemas
</Location>

<LocationMatch "/projetos/[^/]+/login">
        AuthName "Gerenciador de Projetos Trac"
        AuthType Basic
        Require valid-user
        AuthUserFile /etc/apache2/dav_svn.passwd
</LocationMatch>
```
### 5.1. Ativar o host virtual de Trac
**~# a2ensite trac.conf**

## 6. SCRIPTS
### 6.1. Arquivo de vínculos

Deve-se criar um arquivo de vínculo entre usuários e projetos no diretório /usr/local/etc/ com o nome [`svn_authz.conf`](https://github.com/clayton-camargo/FULL-SUBVERSION-SVN/blob/main/svn_authz.conf) seguindo o padrão abaixo:

**~# vi /usr/local/etc/svn_authz.conf**
```
[groups]
admin =  claytoncamargo #Participantes que terão acesso full a todos os projetos#
 
#FIM-GRUPOS
#----------------------------------------------------------------------------------------------------------------

```

### 6.2. Script cria usuários
Para adcionar usuários deve-se criar um script dentro do diretório /root com o nome [`cria-usuarios`](https://github.com/clayton-camargo/FULL-SUBVERSION-SVN/blob/main/cria-usuarios) e adicionar os seguintes paramentos:

**~# vi cria-usuarios**
```
#!/bin/bash

# Script para gerar usuarios e senhas para os sistemas
# Autor: Clayton Camargo
# Data: 06/12/2021

USUARIO=$( dialog --stdout --inputbox 'Digite o nome do usuario(ex.: clayton.camargo)' 0 45 )
SENHA=$( dialog --stdout --inputbox 'Digite a senha(ex.: yourpass#2021)' 0 45 )

htpasswd -mb /etc/apache2/dav_svn.passwd $USUARIO $SENHA
```
**~# chmod +x cria-usuarios**

>_Este comando tonará o script cria-usuarios executável._

>_Para executar o script, no diretório /root digite o comando `./cria-usuarios`. A execução deste script abrirá uma caixa de diálogo no qual deve-se inserir o nome e senha do usuário a ser criado. Ao informar as credenciais do usuário, este será adcionado de forma criptografada ao diretório /etc/apache2/dav_svn.passwd._


### 6.3 Script cria projeto

Agora, para criar os projetos deve-se primeiramente criar um script dentro do diretório /root com o nome [`cria-projeto`](https://github.com/clayton-camargo/FULL-SUBVERSION-SVN/blob/main/cria-projeto) e adicionar os seguintes parâmetros:

**~# vi cria-projeto**
```
#!/bin/bash

# Script para gerar o diretorio svn e trac para projeto
# Autor: CLAYTON CAMARGO

# Dialog para pegar o nome do projeto
PROJETO=$( dialog --stdout --inputbox 'Digite o nome do projeto (ex.: DSV-XPTO)' 0 45 )
NOMETRAC=$( dialog --stdout --inputbox 'Digite o nome do TRAC (ex.: DSV XPTO)' 0 45 )
NOME_INTEGRANTES=$( dialog --stdout --inputbox 'Digite o(s) nome(s) do(s) integrante(s) do Projeto (ex.: Clayton Camargo)' 0 45 )
USER_INTEGRANTES=$( dialog --stdout --inputbox 'Digite o(s) login(s) do(s) integrante(s) do Projeto (ex.: clayton.camargo)' 0 45 )

NOME_GRUPO=${PROJETO,,}
DIR_INI=${DIRETORIO_INICIAL,,}

CAMINHO="/var/lib/svn/$PROJETO"
DB="sqlite:db/trac.db"
REPOSTYPE=svn

# Criando o diretorio de projeto
mkdir -p /var/lib/svn/$PROJETO

touch /var/www/html/integrantes/$PROJETO.html

echo "<!DOCTYPE html>" > /var/www/html/integrantes/$PROJETO.html
echo "<html lang="pt-BR">" >> /var/www/html/integrantes/$PROJETO.html
echo "<head>" >> /var/www/html/integrantes/$PROJETO.html
echo    "<meta charset="UTF-8">" >> /var/www/html/integrantes/$PROJETO.html
echo    "<meta name="viewport" content="width=device-width, initial-scale=1.0">" >> /var/www/html/integrantes/$PROJETO.html
#echo	"<link href='https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css' rel='stylesheet' integrity='sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC' crossorigin='anonymous'>" >> /var/www/html/integrantes/$PROJETO.html 
echo    "<title>PROJETO $PROJETO</title>" >> /var/www/html/integrantes/$PROJETO.html
echo "</head>" >> /var/www/html/integrantes/$PROJETO.html
echo "<body>" >> /var/www/html/integrantes/$PROJETO.html
echo    "<h1>INFORMAÇÕES DO PROJETO $PROJETO</h1>" >> /var/www/html/integrantes/$PROJETO.html
echo    "<style type="text/css">" >> /var/www/html/integrantes/$PROJETO.html
echo        "h1 {" >> /var/www/html/integrantes/$PROJETO.html
echo            "color:blue;" >> /var/www/html/integrantes/$PROJETO.html
echo            "text-transform: uppercase;" >> /var/www/html/integrantes/$PROJETO.html
echo            "width: 900px;" >> /var/www/html/integrantes/$PROJETO.html
echo            "font-size: x-large;" >> /var/www/html/integrantes/$PROJETO.html
echo            "font-family: verdana;" >> /var/www/html/integrantes/$PROJETO.html
echo            "text-align: center;" >> /var/www/html/integrantes/$PROJETO.html
echo            "text-decoration: underline;" >> /var/www/html/integrantes/$PROJETO.html
echo           "}" >> /var/www/html/integrantes/$PROJETO.html
echo    "</style>" >> /var/www/html/integrantes/$PROJETO.html
echo "</head>" >> /var/www/html/integrantes/$PROJETO.html
echo "<body>" >> /var/www/html/integrantes/$PROJETO.html
echo    "<p>## Projeto = $PROJETO</p>" >> /var/www/html/integrantes/$PROJETO.html
echo    "<p>## Integrantes do Projeto $PROJETO = $NOME_INTEGRANTES</p>" >> /var/www/html/integrantes/$PROJETO.html
echo    "<p>## Credenciais dos Integrantes do Projeto $PROJETO = $USER_INTEGRANTES</p>" >> /var/www/html/integrantes/$PROJETO.html
echo         "<style type="text/css">" >> /var/www/html/integrantes/$PROJETO.html
echo        "p {" >> /var/www/html/integrantes/$PROJETO.html
echo           "color:black;" >> /var/www/html/integrantes/$PROJETO.html
#echo            "text-transform: uppercase;" >> /var/www/html/integrantes/$PROJETO.html
echo            "width: 900px;" >> /var/www/html/integrantes/$PROJETO.html
echo            "font-size: 15px;" >> /var/www/html/integrantes/$PROJETO.html
echo            "font-family: arial;" >> /var/www/html/integrantes/$PROJETO.html
echo       "}" >> /var/www/html/integrantes/$PROJETO.html
echo    "</style>" >> /var/www/html/integrantes/$PROJETO.html
echo "</head>" >> /var/www/html/integrantes/$PROJETO.html
echo "</body>" >> /var/www/html/integrantes/$PROJETO.html
echo "</html>" >> /var/www/html/integrantes/$PROJETO.html
echo "<feff>" >> /var/www/html/integrantes/$PROJETO.html
# Colocar entrada no arquivo de permissoes do svn
sed -i "/<!FIM-GRUPOS>/i </li><li>" /var/www/html/integrantes/index.html
sed -i "/<!FIM-GRUPOS>/i <a href=/integrantes/$PROJETO.html title=Projeto>$PROJETO</a>" /var/www/html/integrantes/index.html

# Criando diretorio com um arquivo vazio para importacao inicial
mkdir -p /tmp/$PROJETO
#touch /tmp/$PROJETO/LEIAME.txt

# Colocando o diretorio no SVN
svnadmin create /var/lib/svn/$PROJETO

# Importacao inicial
svn import /tmp/$PROJETO file:///var/lib/svn/$PROJETO -m "Importacao inicial"
rm -rf /tmp/$PROJETO

# Permissoes para o svn
chown -R www-data:www-data /var/lib/svn/$PROJETO
chmod -R 770 /var/lib/svn/$PROJETO

# Criando o trac para o projeto
mkdir -p /var/lib/trac/$PROJETO
trac-admin /var/lib/trac/$PROJETO initenv "$NOMETRAC" $DB $REPOSTYPE $CAMINHO

# Permissoes para o trac
find /var/lib/trac/$PROJETO -type f -exec chmod 660 {} \;
find /var/lib/trac/$PROJETO -type d -exec chmod 2770 {} \;
chown -R www-data.www-data /var/lib/trac/$PROJETO

# Permissao de super-usuario para o administrador de TI
trac-admin /var/lib/trac/$PROJETO permission add claytoncamargo TRAC_ADMIN TICKET_ADMIN

# Permissao de anonymous
trac-admin /var/lib/trac/$PROJETO permission add anonymous BROWSER_VIEW

# Remove permissoes de anonymous
trac-admin /var/lib/trac/$PROJETO permission remove anonymous CHANGESET_VIEW FILE_VIEW LOG_VIEW MILESTONE_VIEW REPORT_SQL_VIEW REPORT_VIEW ROADMAP_VIEW SEARCH_VIEW TICKET_VIEW TIMELINE_VIEW WIKI_VIEW

# Remover milestones padroes
trac-admin /var/lib/trac/$PROJETO milestone remove milestone1
trac-admin /var/lib/trac/$PROJETO milestone remove milestone2
trac-admin /var/lib/trac/$PROJETO milestone remove milestone3
trac-admin /var/lib/trac/$PROJETO milestone remove milestone4

# Alterando as prioridades padroes
trac-admin /var/lib/trac/$PROJETO priority change blocker autissima
trac-admin /var/lib/trac/$PROJETO priority change critical critica
trac-admin /var/lib/trac/$PROJETO priority change major alta
trac-admin /var/lib/trac/$PROJETO priority change minor minima

# Alterando os tipos de ticket
trac-admin /var/lib/trac/$PROJETO ticket_type change defect defeito
trac-admin /var/lib/trac/$PROJETO ticket_type change enhancement etapa
trac-admin /var/lib/trac/$PROJETO ticket_type change task tarefa

# Removendo componentes
trac-admin /var/lib/trac/$PROJETO component remove component1
trac-admin /var/lib/trac/$PROJETO component remove component2

sed -i 25,31d /var/lib/trac/$PROJETO/conf/trac.ini
sed -i '/.type = svn/a .sync_per_request = true' /var/lib/trac/$PROJETO/conf/trac.ini

echo "[header_logo]" >> /var/lib/trac/$PROJETO/conf/trac.ini 
echo "alt = MYCOMP" >> /var/lib/trac/$PROJETO/conf/trac.ini 
echo "height = -5" >> /var/lib/trac/$PROJETO/conf/trac.ini
echo "link = https://www.linkedin.com/in/clayton-camargo-oliveira-463ba9105/" >> /var/lib/trac/$PROJETO/conf/trac.ini
echo "src = https://raw.githubusercontent.com/clayton-camargo/FULL-SUBVERSION-SVN/main/Icone%20clayton.png" >> /var/lib/trac/$PROJETO/conf/trac.ini
echo "width = -5" >> /var/lib/trac/$PROJETO/conf/trac.ini 
echo  >> /var/lib/trac/$PROJETO/conf/trac.ini

echo "[components]" >> /var/lib/trac/$PROJETO/conf/trac.ini
echo "tracopt.versioncontrol.svn.* = enabled" >> /var/lib/trac/$PROJETO/conf/trac.ini 
echo "trac.versioncontrol.api.repositorymanager = enabled" >> /var/lib/trac/$PROJETO/conf/trac.ini
echo "trac.versioncontrol.svn_authz.svnauthzoptions = enabled" >> /var/lib/trac/$PROJETO/conf/trac.ini
echo "trac.versioncontrol.svn_fs.subversionconnector = enabled" >> /var/lib/trac/$PROJETO/conf/trac.ini

# Resincronizando para evitar erros
trac-admin /var/lib/trac/$PROJETO repository sync "*"

# Colocar entrada no arquivo de permissoes do svn
sed -i "/#FIM-GRUPOS/i ## Grupo: $PROJETO ##" /usr/local/etc/svn_authz.conf
sed -i "/#FIM-GRUPOS/i ## $NOME_INTEGRANTES ##" /usr/local/etc/svn_authz.conf
sed -i "/#FIM-GRUPOS/i $NOME_GRUPO = $USER_INTEGRANTES\n" /usr/local/etc/svn_authz.conf

echo "# Repositorio: $PROJETO" >> /usr/local/etc/svn_authz.conf
echo "[$PROJETO:/]" >> /usr/local/etc/svn_authz.conf
echo "@admin = rw" >> /usr/local/etc/svn_authz.conf
echo "@$NOME_GRUPO = rw" >> /usr/local/etc/svn_authz.conf
echo "" >> /usr/local/etc/svn_authz.conf
```

**~# chmod +x cria-projetos**

>_Este comando tonará o script cria-projetos executável._

>_Para executar o script, no diretório /root digite o comando `./cria-projetos`. A execução deste script abrirá uma caixa de diálogo no qual solicitará que seja inserido o nome do projeto a ser criado, assim como o nome de referência para o TRAC, nome(s) e login(s) do(s) usuário(s) pertencentes ao referente projeto. Após a execução do script, será criado o projeto no diretório /var/lib/svn/”nome-projeto” e será adicionado as referências deste projeto no arquivo svn_authz.conf em /usr/local/etc/._

>_•	Pronto, se tudo ocorreu da forma desejada, o SVN está agora configurado e funcionando. Para testar sua efetividade acesse o navegador e insira o seguinte endereço:
http://IPSERVIDOR/svn/NOME-PROJETO_

>_•	Para se testar a funcionalidade do TRAC, acesse o navegador e insira o seguinte endereço:
http://IPSERVIDOR/projetos/NOME-PROJETO_


## 7. BACKUPS PROJETOS SVN E CONFIGURAÇÕES DO SISTEMA
### 7.1. Backup SVN
Agora que o sistema está configurado e funcionando corretamente é necessário configurarmos as rotinas de backup. Para isso, o primeiro passo é criar um script de backup que irá fazer um dump dos projetos SVN e salvá-los em algum dispositivo externo.

Este script tem a função de fazer backup Full para os projetos novos e incremental para os projetos já existente. Por exemplo, a primeira vez que o script for executado, ele criará um backup completo de todos os projetos existentes, e posteriormente criará backups incrementais apenas das modificações de cada projeto. 

Segue abaixo definição do script:

Este script ficará alocado no diretorio /etc/usr/local/bin com o nome 
~# cd /etc/usr/local/bin
