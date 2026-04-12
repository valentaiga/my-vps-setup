# VPS setup

## Open VPN
### Setup
1. `docker run -v ./vps-setup-data/ovpn:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://<your domain>`
2. `docker run -v ./vps-setup-data/ovpn:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki`

### Add user
1. `docker exec -it ovpn easyrsa build-client-full <client name> nopass`
2. `docker exec -it ovpn ovpn_getclient <client name> > ./vps-setup-data/ovpn/pki/issued/<client name>.ovpn`
3. `exit`
4. `scp -r root@<your host>:~/my-vps-setup/vps-setup-data vps-setup-data`
5. `docker compose up -d ovpn`

## MTProxy
### Setup
1. `nano .env (use env.example)`
2. change SECRET (32 chars long)
3. fill EXTERNAL_IP
4. `docker compose up -d MTProxy`

## Dante Socks5
### Setup
1. `nano .env (use env.example)`
2. change DANTE_USER
3. change DANTE_PASS
4. `nano nano vps-setup-data/dante/sockd.conf.example` (save as sockd.conf)

## Hysteria + nginx
### Setup
1. `apt update && apt install certbot -y`
2. `certbot certonly --standalone -d <your domain> --non-interactive --agree-tos --email <your email>`
3. `cp /etc/letsencrypt/live/<your domain>/fullchain.pem ./vps-setup-data/hysteria/acme/<your domain>.crt`
4. `cp /etc/letsencrypt/live/<your domain>/privkey.pem ./vps-setup-data/hysteria/acme/<your domain>.key`
5. `echo "0 0 * * * certbot renew --quiet --post-hook 'cd ~/my-vps-setup && docker-compose restart nginx hysteria'" | crontab -`
6. `nano ./vps-setup-data/hysteria/hysteria.yaml.example` (save as hysteria.yaml)
7. `nano ./vps-setup-data/nginx/nginx.conf.example` (save as nginx.conf)
8. `docker compose up -d nginx hysteria`

