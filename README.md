## Criando a aplicação
Primeiro crie uma aplicação para ser servida, segue aqui um exemplo em NodeJS.

- Start o projeto com o seguinte comando:
````bash
yarn init -y
````

- Instale a dependência:
````bash
yarn add express
````

- Crie o arquivo `index.js` e cole o seguinte código:
````javascript
const express = require('express');
const app = express();
const port = 3333;

// Rota para lidar com a requisição GET na raiz
app.get('/', (req, res) => {
  res.send('<br> <h3>Hello World!<\h3>');
});

// Iniciar o servidor
app.listen(port, () => {
  console.log(`Servidor rodando em http://localhost:${port}`);
});
````

- Agora instala a dependência pm2 que executar a aplicação em segundo plano:
````bash
sudo yarn global add pm2
````

- Disponibilize a aplicação em segundo plano com:
````bash
pm2 start index.js --name test
````


## Proxy reverso - Ngnix
- Execute os seguintes comandos para atualizar a lista de pacotes e instalar o Nginx:
````bash
sudo apt-get update & sudo apt-get install nginx
````

- Após a instalação ser concluída, o Nginx estará em execução e configurado para iniciar automaticamente no boot do sistema.
  1. Iniciar o serviço Nginx:
  ````bash
  sudo systemctl start nginx
  ````

  2. Parar o serviço Nginx:
  ````bash
  sudo systemctl stop nginx
  ````

  3. Reiniciar o serviço Nginx:
  ````bash
  sudo systemctl restart nginx
  ````

- Abra o arquivo de configuração do Nginx usando um editor de texto.
````bash
sudo vi /etc/nginx/sites-available/default
````

- Cole o seguinte código:
````nginx
server {
  listen 80 default_server;
  listen [::]:80 default_server;

  client_max_body_size 100M;

  location / {
    proxy_pass_request_headers on;
    proxy_pass http://$endereco_backend:$port;
  }
}
````

- Reinicie o serviço Nginx para que as alterações entrem em vigor:
````bash
sudo systemctl restart nginx
````
Agora, o Nginx está configurado como um proxy reverso. Todas as solicitações recebidas pelo Nginx serão encaminhadas para o servidor backend especificado na configuração.


## Certificado SSL - Certbot
- Instalando as dependencias
````bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo apt-get update
````

- Instalando o certbot
````bash
sudo apt-get install certbot
````

- Instalando pluing nginx
````bash
sudo apt-get install python3-certbot-nginx
````

- Execute o seguinte comando para obter o certificado SSL usando o Certbot:
````bash
sudo certbot certonly --nginx -d seudominio.com
````

- O Certbot se comunicará com o Let's Encrypt para verificar a propriedade do domínio e emitir o certificado SSL. Após a conclusão bem-sucedida desse processo, o certificado será salvo em um diretório específico no seu servidor.

- O Certbot também configurará automaticamente o arquivo de configuração do Nginx para usar o certificado SSL recém-obtido. Se ele não modificar o arquivo do nginx, cole o seguinte código no `/etc/nginx/sites-available/default`
````nginx
server {
  listen 80 default_server;
  listen [::]:80 default_server;

  client_max_body_size 100M;

  location / {
    proxy_pass_request_headers on;
    proxy_pass http://endereco_backend:port;
  }
}

server {
  listen [::]:443 ssl ipv6only=on;
  listen 443 ssl;

  server_name seudominio.com;

  location / {
    proxy_pass_request_headers on;
    proxy_pass http://$endereco_backend:$port;
  }

  # managed by Certbot --------
  ssl_certificate /etc/letsencrypt/live/seudominio.com/fullchain.pem; 
  ssl_certificate_key /etc/letsencrypt/live/seudominio.com/privkey.pem;

  include /etc/letsencrypt/options-ssl-nginx.conf;
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
  if ($host = seudominio.com) {
    return 301 https://$host$request_uri;
  }

  listen 80;
  listen [::]:80;

  server_name seudominio.com;
  return 404;
}
````

- Reinicie o Nginx para que as alterações de configuração entrem em vigor:
````bash
sudo systemctl restart nginx
````

| Pronto, agora o dominio estará certificado.
