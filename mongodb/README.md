# Instalando o MongoDB no CentOS 7
O MongoDB é um banco de dados orientado a documento, livre e de código aberto. Ele é classificado como um banco de dados NoSQL porque não se baseia na estrutura dos tradicionais banco de dados relacionais, que é baseada em tabela.

## Passo 01 - Adicione o repositório do MongoDB

Crie um arquivo `.repo`:

    sudo vi /etc/yum.repos.d/mongodb-org.repo

Adicione o seguinte conteúdo ao arquivo:

    [mongodb-org-3.2]
    name=MongoDB Repository
    baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
    gpgcheck=1
    enabled=1
    gpgkey=https://www.mongodb.org/static/pgp/server-3.2.asc

Verifique sempre a [documentação do MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/#configure-the-package-management-system-yum) para obter as informações do repositório com a última versão instável.

## Passo 02 - Instale o MongoDB

Para continuar com a intalação, execute o seguinte comando:

    sudo yum install mongodb-org

Quando solicitado, pressione `y` para confirmar a instalação e depois para comfirmar a chave GPG.

## Comandos básicos

Para iniciar o serviço do MongoDB:

    sudo systemctl start mongod

Para parar o serviço do MongoDB:

    sudo systemctl stop mongod

Para reiniciar o serviço:

    sudo systemctl restart mongod

Para recarregar alguma configuração feita no arquivo `/etc/mongod.conf` sem reiniciar o serviço:

    sudo systemctl reload mongod

Para verificar o log:

    sudo tail -f /var/log/mongodb/mongod.log

Para acessar o shell do MongoDB:

    mongo

## Fazendo alguns ajustes

Caso o seu log exiba a seguinte mensagem:

    ** WARNING: soft rlimits too low. rlimits set to 4096 processes, 64000 files.
    Number of processes should be at least 32000 : 0.5 times number of files.

Edite o arquivo `/etc/security/limits.d/20-nproc.conf` e adicione o seguinte conteúdo no final:

    mongod soft nproc 32000

Feito isso, reinicie o serviço:

    sudo systemctl restart mongod

Caso seu log exiba a seguinte mensagem:

    ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
    **        We suggest setting it to 'never'

Proceda conforme descrito [aqui](https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/)

## Referências
[MongoDB: Manual](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)

[DigitalOcean: How To Install MongoDB on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-centos-7)

[MongoDB: Disable Transparent Huge Pages (THP)](https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/)
