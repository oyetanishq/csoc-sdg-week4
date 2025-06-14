# week 4

For week 3 I submitted hackpad, which was a full-stack project for coders who wants to code together,
to put that in nginx, I divided the backend in two services:

1. auth
2. project

but as the deployments cost $, I deployed both to vercel, and hosted a droplet from digital ocean, installed nginx in it, and reversed proxied to those vercel servers

#### this conf is what deployed in droplet, but it's not how its dont, i just did to save $

its live you can check here [http://dudewtf.tanishqsingh.com](http://dudewtf.tanishqsingh.com)
the ssl warning that you'll receive is because it's self signed using openssl (please ignore).

```nginx
events {}

http {
    upstream auth_backend {
        server hackpad-backend.vercel.app;
    }

    upstream project_backend {
        server hackpad-backend.vercel.app;
    }

    server {
        listen 443 ssl;
        server_name dudewtf.tanishqsingh.com;

        ssl_certificate     /root/ssl/server.crt;
        ssl_certificate_key /root/ssl/server.key;

        location /auth/ {
            proxy_pass https://auth_backend/auth/;
        }

        location /projects/ {
            proxy_pass https://project_backend/project/;
        }

        location / {
            proxy_pass https://hackpad.vercel.app;
        }
    }

    server {
        listen 80;
        server_name dudewtf.tanishqsingh.com;
        return 301 https://$host$request_uri;
    }
}
```

#### this is how it should be done

each request is send to main server which then works as a reverse proxy and transfer request to different physical servers, this is horizontal scaling,
other type of scaling is vertical scaling where we increase the architecture of the single physical host like ram, storage, cpu core etc.
```nginx
events {}

http {
    upstream auth_backend {
        # these all are different physical machines
        server hackpad-backend-auth-1.vercel.app;
        server hackpad-backend-auth-2.vercel.app;
        server hackpad-backend-auth-3.vercel.app;
    }

    upstream project_backend {
        server hackpad-backend-project-1.vercel.app;
        server hackpad-backend-project-2.vercel.app;
        server hackpad-backend-project-3.vercel.app;
    }

    upstream frontend {
        server hackpad-1.vercel.app;
        server hackpad-2.vercel.app;
        server hackpad-3.vercel.app;
    }

    server {
        listen 443 ssl;
        server_name dudewtf.tanishqsingh.com;

        ssl_certificate     /root/ssl/server.crt;
        ssl_certificate_key /root/ssl/server.key;

        location /auth/ {
            proxy_pass https://auth_backend;
        }

        location /project/ {
            proxy_pass https://project_backend;
        }

        location / {
            proxy_pass https://frontend;
        }
    }

    server {
        listen 80;
        server_name dudewtf.tanishqsingh.com;
        return 301 https://$host$request_uri;
    }
}
```
