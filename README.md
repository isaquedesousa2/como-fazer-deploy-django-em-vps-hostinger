# COMO FAZER DEPLOY DJANGO EM VPS HOSTIGER

# Abra seu prompt de comando

## 1 - Conecte-se a vps

```
ssh root@<ip_da_vps>
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
sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl
```
```
sudo apt install certbot python3-certbot-nginx -y
```
```
sudo apt install git
```

## 3 - Criando o banco de dados e o usuário PostgreSQL

```
sudo -u postgres psql
```
## Use sempre o ; no final de cada comando
### Crie o banco de dados

```
CREATE DATABASE nomedoprojeto;
```
### Crie um usuário de banco de dados
```
CREATE USER usuário WITH PASSWORD 'senha';
```
### Modifique alguns parâmetros de conexão para o usuário que acabamos de criar
```
ALTER ROLE usuário SET client_encoding TO 'utf8';
```
```
ALTER ROLE usuário SET default_transaction_isolation TO 'read committed';
```
```
ALTER ROLE usuário SET timezone TO 'UTC';
```