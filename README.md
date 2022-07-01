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
sudo apt install postgresql postgresql-contrib nginx curl
```
```
sudo apt install certbot python3-certbot-nginx -y
```
```
sudo apt-get install ufw
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
CREATE DATABASE <nomedobanco>;
```
### Crie um usuário de banco de dados
```
CREATE USER <usuário> WITH PASSWORD '<senha>';
```
### Modifique alguns parâmetros de conexão para o usuário que acabamos de criar
```
ALTER ROLE <usuário> SET client_encoding TO 'utf8';
```
```
ALTER ROLE <usuário> SET default_transaction_isolation TO 'read committed';
```
```
ALTER ROLE <usuário> SET timezone TO 'UTC';
```
### Agora vamos da acesso ao novo usuário para administrar o novo banco de dados
```
GRANT ALL PRIVILEGES ON DATABASE <nomedobanco> TO <usuário>;
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
git config --global user.name '<nome>'
```
```
git config --global user.email '<email>'
```
```
git config --global init.defaultBranch main
```

# 5 - Crie um repositório no servidor 

### Um repositório bare é um repositório transitório (como se fosse um github).

```
mkdir -p ~/app_bare_<nomedoprojeto>
```
```
cd ~/app_bare_<nomedoprojeto>
```
```
git init --bare
```
```
cd ..
```
### Crie o repositório da aplicação
```
mkdir -p ~/app_repo_<nomedoprojeto>
```
```
cd ~/app_repo_<nomedoprojeto>
```
```
git init
```
```
git remote add origin ~/app_bare_<nomedoprojeto>
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
git remote add app_bare_<nomedoprojeto> root@ip_da_vps:~/app_bare_<nomedoprojeto>
```
```
git push app_bare_<nomedoprojeto> <branch>
```
### No servidor, em app_repo, faça pull:
```
cd ~/app_repo_<nomedoprojeto>
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
pip install psycopg2-binary
```
```
pip install gunicorn
```

### Agora se tudo deu certo quando rodamos esse comando é para o servidor subir
```
python manage.py runserver
```

# 7 - Conectar o banco de dados com a aplicação

### Crie suas variaveis de ambiente e salve em um arquivo, por exemplo .env-example

### Suba esse arquivo para o servidor e faça esses comandos

```
cd ~/app_repo_<nomedoprojeto>
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

### Subistitua

```
<gunicorn> para o nome do arquivo gunicorn que você deseja
```
### Você pode conferir seu nome de usuário com esse comando
```
whoami
```
```
<user> para seu nome de usuário
```
### No caso seria app_repo_nomedoprojeto
```
<projeto> para o nome da pasta do seu projeto repo
```
### Seria a pasta do seu projeto django
```
<wsgi> para o nome da pasta onde você encontra um arquivo chamado wsgi.py
```


### Crie o arquivo <gunicorn>.socket
```
sudo nano /etc/systemd/system/<gunicorn>.socket
```
### Coloque esse conteúdo e salve

```
[Unit]
Description=gunicorn blog socket

[Socket]
ListenStream=/run/<gunicorn>.socket

[Install]
WantedBy=sockets.target
```

### Crie o arquivo <gunicorn>.service
```
sudo nano /etc/systemd/system/<gunicorn>.service
```
### Coloque esse conteúdo e salve

```
[Unit]
Description=gunicorn daemon
Requires=<gunicorn>.socket
After=network.target

[Service]
User=<user>
Group=www-data
WorkingDirectory=/<user>/<projeto>
ExecStart=/<user>/<projeto>/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/<projeto>.socket \
          <wsgi>.wsgi:application

[Install]
WantedBy=multi-user.target
```

### Agora vamos ativar
```
sudo systemctl start <gunicorn>.socket
```
```
sudo systemctl enable <gunicorn>.socket
```

### Verifique se tudo deu certo
```
sudo systemctl status <gunicorn>.socket
```
```
sudo systemctl status <gunicorn>.service
```
```
sudo systemctl status <gunicorn>

```
### Verifique a existência do gunicorn.sock arquivo dentro do /run
```
file /run/<gunicorn>.sock
```

### Execute esse comando para enviar uma conexção ao socket curl, em seguir você deve receber a saída HTML do seu aplicativo no terminal
```
curl --unix-socket /run/<gunicorn>.socket localhost
```

### Se o systemctl status ou curl indicou erro você pode usar esse comando para saber o que aconteceu 
```
sudo journalctl -u <gunicorn>.sock
```

### Se precisar reiniciar
```
sudo systemctl restart <gunicorn>.service
```
```
sudo systemctl restart <gunicorn>.socket
```
```
sudo systemctl restart <gunicorn>
```

### Se você precisar alterar algum arquivo rode esse comando para recarega os arquivos
```
sudo systemctl daemon-reload
```
```
sudo systemctl restart <gunicorn>
```

### Para debugar
```
sudo journalctl -u <gunicorn>.service
```
```
sudo journalctl -u <gunicorn>.socket
```

# 9 - Confugurando o Nginx

### Subistitua

### Substitua pelo seu domínio
```
<dominio>
```
### Substitua pelo caminho para a pasta de arquivos estáticos
```
<static>
```
### Substitua pelo caminho para a pasta de arquivos de mídia
```
<media> 
```
### Substitua pelo nome do arquivo <gunicorn>.socket que foi criado
```
<gunicorn>
```
### Agora crie o arquivo nginx e cole o o código a cima depois de subistituir
```
sudo nano /etc/nginx/sites-available/<nomedoarquivo>
```
```
server {
    listen 80;
    server_name <dominio>;

    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }

    location /static/ {
        root <static>;
    }

    location /media/ {
        root <static>;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/<gunicorn>.socket;
    }

}
```



### Excluar o arquivo default
```
sudo rm /etc/nginx/sites-available/default
```
###  Agora, vamos habilitar o arquivo vinculando-o ao sites-enabled
```
sudo ln -s /etc/nginx/sites-available/<nomedoarquivo> /etc/nginx/sites-enabled/
```
### Precisamos abrir nosso firewall para o tráfego normal na porta 80
```
sudo ufw allow 'Nginx Full'
```
### Teste se tudo deu certo 
```
sudo nginx -t
```
### Agora reiniciei o nginx
```
sudo systemctl restart nginx
```

### Se precisar mudar o timezone do servidor e depois reiniciei o servidor
```
sudo timedatectl set-timezone <timezone>
```
```
sudo reboot
```

# 10 - Configurando as collectstatic

```
gpasswd -a www-data username
```
```
chmod g+x /username && chmod g+x /username/test && chmod g+x /username/test/static
```
```
nginx -s reload
```