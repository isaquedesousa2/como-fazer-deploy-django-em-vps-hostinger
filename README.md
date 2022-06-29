# COMO FAZER DEPLOY DJANGO EM VPS HOSTIGER

# Abra seu prompt de comando

## 1 - Conecte-se a vps

```
ssh root@ip_da_vps
```

## 2 - Instalando os pacotes e atualizções necessárias

```
sudo apt update -y
```
```
sudo apt upgrade -y
```
```
sudo apt autoremove -y
```
```
sudo apt install build-essential -y
```
```
sudo apt install python3-pip python3-dev libpq-dev python3-venv
```
```
postgresql postgresql-contrib nginx curl
```
```
sudo apt install certbot python3-certbot-nginx -y
```
```
sudo apt install git
```

## 3 - Criando o banco de dados e o usuário PostgreSQL
### Faça login em uma sessão interativa do Postgres digitando

```
sudo -u postgres psql
```
## Use sempre o ; no final de cada comando
### Crie o banco de dados

```
CREATE DATABASE nomedobanco;
```
### Crie um usuário de banco de dados
```
CREATE USER nomedousuário WITH PASSWORD 'suasenha';
```
### Modifique alguns parâmetros de conexão para o usuário que acabamos de criar
```
ALTER ROLE nomedousuário SET client_encoding TO 'utf8';
```
```
ALTER ROLE nomedousuário SET default_transaction_isolation TO 'read committed';
```
```
ALTER ROLE nomedousuário SET timezone TO 'UTC';
```
### Agora vamos da acesso ao novo usuário para administrar o novo banco de dados
```
GRANT ALL PRIVILEGES ON DATABASE nomedobanco TO nomedousuário;
```
### Saia do prompt do PostgreSQL
```
\q
```
### Reinicie o postgres
```
sudo systemctl restart postgresql
```

### 4 - Configure o git

```
git config --global user.name 'Seu nome'
```
```
git config --global user.email 'seu_email@gmail.com'
```
```
git config --global init.defaultBranch main
```

# 5 - Crie um repositório no servidor 

### Um repositório bare é um repositório transitório (como se fosse um github).

```
mkdir -p ~/app_bare_nomedoprojeto
```
```
cd ~/app_bare
```
```
git init --bare
```
```
cd ..
```
### Crie o repositório da aplicação
```
mkdir -p ~/app_repo_nomedoprojeto
```
```
cd ~/app_repo_nomedoprojeto
```
```
git init
```
```
git remote add origin ~/app_bare_nomedoprojeto
```
```
git add . 
```
```
git commit -m 'Initial'
```
```
cd ..
```
### No seu computador local, adicione o bare como remoto
```
git remote add app_bare_nomedoprojeto root@ip_da_vps:~/app_bare_nomedoprojeto
```
```
git push app_bare <branch>
```
### No servidor, em app_repo, faça pull:
```
cd ~/app_repo_nomedoprojeto
```
```
git pull origin <branch>
```

# 6 - Crie um ambiente virtual

```
python3 -m venv venv
```
```
. venv/bin/activate
```
```
pip install -r requirements.txt
```
```
pip install psycopg2 psycopg2-binary
```
### Se de erro ao instalar o psycopg2 executer esses comandos
```
pip uninstall psycopg2
```
```
pip list --outdated
```
```
pip install --upgrade wheel
```
```
pip install --upgrade setuptools
```
```
pip install psycopg2
```
### Pronto agora é para ter dado tudo certo na instalação do psycopg2
```
pip install gunicorn
```

## Agora se tudo deu certo quando rodamos esse comando é para o servidor subir
```
python manage.py runserver
```

# 7 - conectar o banco de dados com a aplicação

### Crie suas variaveis de ambiente e salve em um arquivo, por exemplo .env-example

### Suba esse arquivo para o servidor e faça esses comandos

```
cd ~/app_repo_nomedoprojeto
```
```
cp .env-example .env
```
```
nano .env
```
### Com a aplicação conectada ao postgres faça as migrações
```
python manage.py migrate
```
```
python manage.py runserver
```

# 8 - Vamos criar agora o unix.socket e configurar o gunicorn

###############################################################################
# Replace
# ___GUNICORN_FILE_NAME___ to the name of the gunicorn file you want
# __YOUR_USER__ to your user name
# __PROJECT_FOLDER__ to the folder name of your project
# __WSGI_FOLDER__ to the folder name where you find a file called wsgi.py
#
###############################################################################
# Criando o arquivo ___GUNICORN_FILE_NAME___.socket
sudo nano /etc/systemd/system/___GUNICORN_FILE_NAME___.socket

###############################################################################
# Conteúdo do arquivo
[Unit]
Description=gunicorn blog socket

[Socket]
ListenStream=/run/___GUNICORN_FILE_NAME___.socket

[Install]
WantedBy=sockets.target

###############################################################################
# Criando o arquivo ___GUNICORN_FILE_NAME___.service
sudo nano /etc/systemd/system/___GUNICORN_FILE_NAME___.service

###############################################################################
# Conteúdo do arquivo
[Unit]
Description=Gunicorn daemon (You can change if you want)
Requires=___GUNICORN_FILE_NAME___.socket
After=network.target

[Service]
User=__YOUR_USER__
Group=www-data
Restart=on-failure
EnvironmentFile=/__YOUR_USER__/__PROJECT_FOLDER__/.env
WorkingDirectory=/__YOUR_USER__/__PROJECT_FOLDER__

ExecStart=/__YOUR_USER__/__PROJECT_FOLDER__/venv/bin/gunicorn \
          --error-logfile /__YOUR_USER__/__PROJECT_FOLDER__/gunicorn-error-log \
          --enable-stdio-inheritance \
          --log-level "debug" \
          --capture-output \
          --access-logfile - \
          --workers 6 \
          --bind unix:/run/___GUNICORN_FILE_NAME___.socket \
          __WSGI_FOLDER__.wsgi:application

[Install]
WantedBy=multi-user.target

###############################################################################
# Ativando
sudo systemctl start ___GUNICORN_FILE_NAME___.socket
sudo systemctl enable ___GUNICORN_FILE_NAME___.socket

# Checando
sudo systemctl status ___GUNICORN_FILE_NAME___.socket
curl --unix-socket /run/___GUNICORN_FILE_NAME___.socket localhost
sudo systemctl status ___GUNICORN_FILE_NAME___

# Reiniciando
sudo systemctl restart ___GUNICORN_FILE_NAME___.service
sudo systemctl restart ___GUNICORN_FILE_NAME___.socket
sudo systemctl restart ___GUNICORN_FILE_NAME___

# After changing something
sudo systemctl daemon-reload

# Debugging
sudo journalctl -u ___GUNICORN_FILE_NAME___.service
sudo journalctl -u ___GUNICORN_FILE_NAME___.socket