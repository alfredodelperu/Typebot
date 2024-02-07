# Typebot: pasos para instalacion de typebot 2024-02-06
  sudo apt update -y
  sudo apt upgrade -y

#editar el archivo rules.v4 para abrir puertos
  sudo vim /etc/iptables/rule.v4
  sudo uptables-restore < /etc/iptables/rules.v4

#verificar que puertos 80 y 443 esten liberados porque seran usado por docker y su servidor integrado
  sudo netstat -tunlp

#Instalr Docker segun instrucciones den pagina web https://docs.docker.com/engine/install/ubuntu/

#Comprobar instalacion
  sudo docker compose version

# Typebot instruciones : https://docs.typebot.io/self-hosting/deploy/docker
#Antes de iniciar la instalacion de Typebot, edital el archivo docker-compos.yml segun instrucciones de https://docs.typebot.io/self-hosting/configuration
   wget https://raw.githubusercontent.com/baptisteArno/typebot.io/latest/docker-compose.yml
   wget https://raw.githubusercontent.com/baptisteArno/typebot.io/latest/.env.example -O .env

--------------- docker-compose.yml -----------------------
version: '3.3'

services:
  caddy-gen:
    image: 'wemakeservices/caddy-gen:latest'
    restart: always
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - caddy-certificates:/data/caddy
    ports:
      - '80:80'
      - '443:443'
    depends_on:
      - typebot-builder
      - typebot-viewer
  typebot-db:
    image: postgres:14-alpine
    restart: always
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=typebot
      - POSTGRES_PASSWORD=typebot
  typebot-builder:
    labels:
      virtual.host: 'tpb.uvdtfperu.com' # change to your domain
      virtual.port: '3000'
      virtual.tls-email: 'juan@uvdtfperu.com' # change to your email
    image: baptistearno/typebot-builder:latest
    restart: always
    depends_on:
      - typebot-db
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    # See https://docs.typebot.io/self-hosting/configuration for more configuration options
    env_file:
      - .env
  typebot-viewer:
    labels:
      virtual.host: 'b.uvdtfperu.com' # change to your domain
      virtual.port: '3000'
      virtual.tls-email: 'juan@uvdtfperu.com' # change to your email
    image: baptistearno/typebot-viewer:latest
    restart: always
    # See https://docs.typebot.io/self-hosting/configuration for more configuration options
    env_file:
      - .env
  mail:
    image: bytemark/smtp
    restart: always
  minio:
    labels:
      virtual.host: 'storage.uvdtfperu.com' # change to your domain
      virtual.port: '9000'
      virtual.tls-email: 'juan@uvdtfperu.com' # change to your email
    image: minio/minio
    command: server /data
    ports:
      - '9000:9000'
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio123
    volumes:
      - s3-data:/data
  # This service just make sure a bucket with the right policies is created
  createbuckets:
    image: minio/mc
    depends_on:
      - minio
    entrypoint: >
      /bin/sh -c "
      sleep 10;
      /usr/bin/mc config host add minio http://minio:9000 minio minio123;
      /usr/bin/mc mb minio/typebot;
      /usr/bin/mc anonymous set public minio/typebot/public;
      exit 0;
      "
volumes:
  db-data:
  s3-data:
  caddy-certificates:
    driver: local

------------------------------------------------------

   
------------------- .env --------------------------------------------------------------------
# Make sure to change this to your own random string of 32 characters (https://docs.typebot.io/self-hosting/docker#2-add-the-required-configuration)
ENCRYPTION_SECRET=ORnXlVatMxp/6Ds6nDrWrtNXCG3I86j4
DATABASE_URL=postgresql://postgres:typebot@typebot-db:5432/typebot
NEXTAUTH_URL=https://tpb.uvdtfperu.com
NEXT_PUBLIC_VIEWER_URL=https://b.uvdtfperu.com
ADMIN_EMAIL=juan@uvdtfperu.com
# For more configuration options check out: https://docs.typebot.io/self-hosting/configuration
DEFAULT_WORKSPACE_PLAN=UNLIMITED
DISABLE_SIGNUP=false
DEBUG=false
NEXT_PUBLIC_BOT_FILE_UPLOAD_MAX_SIZE=10
#Email (Auth, notifications)
SMTP_USERNAME=juan@uvdtfperu.com
SMTP_PASSWORD=Fullcolor2009
SMTP_HOST=mail.uvdtfperu.com
SMTP_PORT=465
NEXT_PUBLIC_SMTP_FROM=juan@uvdtfperu.com
SMTP_SECURE=true
SMTP_AUTH_DISABLED=false
#S3 Storage (Media uploads)
S3_ACCESS_KEY=minio #cambio si es necesario
S3_SECRET_KEY=minio123 #cambio si es necesario
S3_BUCKET=typebot
S3_ENDPOINT=storage.uvdtfperu.com
 
------------------------------------------------------------------------------------
Ejecutar:
sudo docker compose up -d

#Si ocurren problemas con la creacion de iptables, reiniciar el servicio docker
sudo systemctl restart docker

#abrir la direccion tpb.uvdtfperu.com







