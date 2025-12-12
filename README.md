# CI/CD con GitHub Actions y despliegue por SSH

Este repo contiene una app Node.js mínima y un flujo de GitHub Actions que se conecta por SSH a un servidor Ubuntu para hacer `git pull`, instalar dependencias y reiniciar la app con `pm2`.

## 1. Requisitos previos
- Servidor Ubuntu con acceso SSH.
- Node.js + npm instalados en el servidor.
- `git` y `pm2` instalados en el servidor (`npm i -g pm2`).
- Repo en GitHub con branch `main`.

## 2. Generar y registrar la llave SSH
1. En tu máquina local: `ssh-keygen -t ed25519 -C "github-deploy"` y deja la clave en `~/.ssh/id_ed25519`.
2. Copia la clave pública al servidor: `ssh-copy-id -i ~/.ssh/id_ed25519.pub usuario@host` (o pega el contenido en `~/.ssh/authorized_keys`).
3. Prueba el acceso: `ssh usuario@host`.

## 3. Preparar el servidor
```bash
# instala dependencias básicas
a sudo apt update && sudo apt install -y git curl
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm i -g pm2

# clonar la app en el directorio destino (APP_DIR)
cd /var/www
sudo git clone git@github.com:TU_USUARIO/TU_REPO.git my-app
cd my-app
npm install --production
pm2 start server.js --name my-app
pm2 save
pm2 startup systemd -u $USER --hp $HOME
```

## 4. Configurar secretos en GitHub
En *Settings > Secrets and variables > Actions* crea estos secretos:
- `SSH_HOST`: IP o dominio del servidor.
- `SSH_USERNAME`: usuario SSH.
- `SSH_PRIVATE_KEY`: contenido de tu clave privada (id_ed25519).
- `SSH_PORT` (opcional, default 22).
- `APP_DIR`: ruta absoluta al repo en el servidor (ej. `/var/www/my-app`).

## 5. Workflow de deploy (`.github/workflows/deploy.yml`)
Se ejecuta en cada `push` a `main` y:
1) Usa la llave SSH privada para conectarse al servidor.
2) Corre `git pull` en `APP_DIR`.
3) Instala dependencias (`npm install --production`).
4) Reinicia la app con `pm2 restart my-app` (o la inicia si no existe).

## 6. Uso local
```bash
npm install
npm start
# abre http://localhost:3000
```

## 7. Desencadenar el deploy
Haz commit y push a `main`. El workflow se encargará del resto.
