# demo-tinyisv-infra
This repo has info about simple infr for demo website. 

Main component of infra are 
- A linux based VPS that has adocker container. 
- DNS entries to the server's public ip
- A docker network so that traffic can be easily routed
  ```
  docker network create --driver bridge example-net
  ```

- An [nginx](https://github.com/JonasAlfredsson/docker-nginx-certbot) image that can automatically get lets encrypt certificates 
  ```
  docker run -it -p 80:80 -p 443:443  --env CERTBOT_EMAIL=your@email.org --network example-net  -v $(pwd)/nginx_secrets:/etc/letsencrypt  -v $(pwd)/user_conf.d:  /etc/nginx/user_conf.d:ro  --name nginx-certbot jonasal/nginx-certbot:latest
  ```

- Nginx can be configured as follows 
 ```
 server {
    # Listen to port 443 on both IPv4 and IPv6.
    listen 443 ssl default_server reuseport;
    listen [::]:443 ssl default_server reuseport;

    # Domain names this server should respond to.
    server_name tinyisv.com demo.tinyisv.com;

    # Load the certificate files.
    ssl_certificate         /etc/letsencrypt/live/test-name/fullchain.pem;
    ssl_certificate_key     /etc/letsencrypt/live/test-name/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/test-name/chain.pem;

    # Load the Diffie-Hellman parameter.
    ssl_dhparam /etc/letsencrypt/dhparams/dhparam.pem;


    location /api/ {
     proxy_buffering off;
     proxy_pass http://api:8080/;
    }

}
  ```
