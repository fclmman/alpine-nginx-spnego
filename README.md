Docker image with nginx and spnego negotiate auth module built in

Pull

```
docker pull fclmman/alpine-nginx-spnego
```

Usage:

For you to start with kerberos auth some manipulations are expected.
Lets assume your host computer has a "host-linux" hostname, and your domain is "DOMAIN.LOCAL"

You will need do create a keytab file for your host computer. To do so you need to add a computer with "host-linux" to AD.
Next, contact your domain controller administrator to create a SPN and generate a ketab file for you.

The command will look like 

```
C:\Windows\system32>ktpass -princ HTTP/HOST-LINUX.domain.local@DOMAIN.LOCAL -mapuser host-linux-user@DOMAIN.LOCAL -pass yourpassword -cryptoAll -ptype KRB5_NT_PRINCIPAL -out C:\Temp\web.keytab
```

Once you got your keytab we can start with the container. I will provide an example with docker-compose.yml

```
version: "2"
services:
    nginx-spnego:
        image: fclmman/alpine-nginx-spnego
#map ports you will need
        ports:
            - 80:80
            - 5010:5010
            - 443:443
            - 8001:8001
#add volume with the keytab file
        volumes:
            - ./config:/home/spnego/config
```

Your nginx.conf and web.keytab should be in the ./config volume, as the config path is set to /home/spnego/config and ketab is also should be available in container.
Your nginx.conf will look like

```
http {
    #Whatever is there by default

    server {
        listen       80;
        server_name  localhost;
        
		#Here kerberos stuff starts
		auth_gss     on;
        auth_gss_realm DOMAIN.LOCAL;
		#Keytab file from the mounted folder
        auth_gss_keytab /home/spnego/config/web.keytab;
        auth_gss_service_name HTTP/HOST-LINUX.domain.local;
        auth_gss_allow_basic_fallback off;
		#Here kerberos stuff ends
		
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

```

That's all. But you should know that kerberos is not working by IP, it supports hostname only