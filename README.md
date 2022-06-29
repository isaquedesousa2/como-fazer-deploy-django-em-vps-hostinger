# COMO FAZER DEPLOY DJANGO EM VPS HOSTIGER

# Abra seu prompt de comando

## 1 - Conecte-se a vps

```
ssh root@<ip_da_vps>
```

## 2 - Agora vamos instalar os pacotes e atualizções necessárias

```
sudo apt update -y
```
sudo apt upgrade -y
```
sudo apt autoremove -y
```
sudo apt install build-essential -y
```
sudo apt install python3.9 python3.9-venv python3.9-dev -y
```
sudo apt install nginx -y
```
sudo apt install certbot python3-certbot-nginx -y
```
sudo apt install postgresql postgresql-contrib -y
```
sudo apt install libpq-dev -y
```
sudo apt install git
```