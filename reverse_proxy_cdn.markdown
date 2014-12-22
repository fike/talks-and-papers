Algumas empresas por diversas razões tem dificuldade para usar uma [CDN (Content Delivery Network)][]. Uma dessas situações é quando ele não tem um balanceador de carga ou proxy reverso para *HTTP/HTTPS*. Para disponibilizar um site na internet geralmente usa-se uma porta *TCP* diferente da **80** e **443**. Por exemplo:

- http://www.site.com/
- http://extranet.site.com:8008/
- https://www.site.com/8080/crm/


{% img center /images/sem_proxy.png %}


    Na figura acima tem três sites em datacenter. 

**Site** - O “Site” está disponível na internet pelo endereço de rede (IP) **200.200.200.200**, ele está funcionando na porta *TCP* **80** e com o nome (domínio/hostname) de www.site.com. Sendo que na rede interna da empresa está disponível no endereço de rede **10.0.20.17** e na porta **TCP 80**.

**Extranet** - A Extranet está disponível na internet pelo endereço de rede (IP) 200.200.200.201, ele está funcionando no port TCP 8008 e com o nome (domínio/hostname) de extranet.site.com. Sendo que na rede interna da empresa está disponível no endereço de rede 10.0.20.37 e na porta TCP 8008.

**CRM** - O CRM está disponível na internet pelo endereço de rede (IP) 200.200.200.201, ele está funcionando no port TCP 8080 e com o nome (domínio/hostname) de extranet.site.com. Sendo que na rede interna da empresa está disponível no endereço de rede 10.0.20.31, na porta TCP 8080 e no subdiretório /crm. Por fim, o CRM usa HTTPS.

       
<pre>
root@proxy1:~# aptitude install nginx
The following NEW packages will be installed:
  libgd2-noxpm{a} libjpeg8{a} libpng12-0{a} libxslt1.1{a} nginx 
  nginx-common{a} nginx-full{a} 
0 packages upgraded, 7 newly installed, 0 to remove and 2 not upgraded.
Need to get 1.376 kB of archives. After unpacking 2.929 kB will be used.
Do you want to continue? [Y/n/?] 
Get: 1 http://ftp.br.debian.org/debian/ wheezy/main libjpeg8 i386 8d-1 [132 kB]
Get: 2 http://ftp.br.debian.org/debian/ wheezy/main libpng12-0 i386 1.2.49-1 [192 kB]
Get: 3 http://ftp.br.debian.org/debian/ wheezy/main libgd2-noxpm i386 2.0.36~rc1~dfsg-6.1 [229 kB]
Get: 4 http://ftp.br.debian.org/debian/ wheezy/main libxslt1.1 i386 1.1.26-14.1 [251 kB]
Get: 5 http://ftp.br.debian.org/debian/ wheezy/main nginx-common all 1.2.1-2.2 [72,9 kB]
Get: 6 http://ftp.br.debian.org/debian/ wheezy/main nginx-full i386 1.2.1-2.2 [436 kB]
Get: 7 http://ftp.br.debian.org/debian/ wheezy/main nginx all 1.2.1-2.2 [60,9 kB] 
Fetched 1.376 kB in 6s (215 kB/s)                                                 
Selecting previously unselected package libjpeg8:i386.
(Reading database ... 48620 files and directories currently installed.)
Unpacking libjpeg8:i386 (from .../libjpeg8_8d-1_i386.deb) ...
Selecting previously unselected package libpng12-0:i386.
Unpacking libpng12-0:i386 (from .../libpng12-0_1.2.49-1_i386.deb) ...
Selecting previously unselected package libgd2-noxpm:i386.
Unpacking libgd2-noxpm:i386 (from .../libgd2-noxpm_2.0.36~rc1~dfsg-6.1_i386.deb) ...
Selecting previously unselected package libxslt1.1:i386.
Unpacking libxslt1.1:i386 (from .../libxslt1.1_1.1.26-14.1_i386.deb) ...
Selecting previously unselected package nginx-common.
Unpacking nginx-common (from .../nginx-common_1.2.1-2.2_all.deb) ...
Selecting previously unselected package nginx-full.
Unpacking nginx-full (from .../nginx-full_1.2.1-2.2_i386.deb) ...
Selecting previously unselected package nginx.
Unpacking nginx (from .../nginx_1.2.1-2.2_all.deb) ...
Processing triggers for man-db ...
Setting up libjpeg8:i386 (8d-1) ...
Setting up libpng12-0:i386 (1.2.49-1) ...
Setting up libgd2-noxpm:i386 (2.0.36~rc1~dfsg-6.1) ...
Setting up libxslt1.1:i386 (1.1.26-14.1) ...
Setting up nginx-common (1.2.1-2.2) ...
Setting up nginx-full (1.2.1-2.2) ...
Setting up nginx (1.2.1-2.2) ...
                                         
root@proxy1:~# 
</pre>

#cat /etc/nginx/sites-avaliable/www.site.com
server {
       listen 80;
       server_name site.com www.site.com ;
       if ($http_host != "www.site.com") {
                 rewrite ^ http://www.site.com$request_uri permanent;
       }

       location / {
                proxy_pass http://10.0.20.17/;
                include /etc/nginx/proxy_params;
       }
}

#cat /etc/nginx/sites-avaliable/extranet.site.com
server {
       listen 80;
       server_name extranet.site.com;
       if ($http_host != "extranet.site.com") {
                 rewrite ^ https://extranet.site.com$request_uri permanent;
       }
}

server {
       listen 443;
       server_name extranet.site.com;
       if ($http_host != "extranet.site.com") {
                 rewrite ^ https://extranet.site.com$request_uri permanent;
       }
       ssl on;
       ssl_certificate /etc/nginx/ssl/extranet.site.com.crt;
       ssl_certificate_key /etc/nginx/ssl/extranet.site.com.key;

       location / {
                proxy_pass http://10.0.20.37/;
                include /etc/nginx/proxy_params;
       } 
       location /crm {
                proxy_pass http://10.0.20.31/;
                include /etc/nginx/proxy_params;
       } 
}

#ln -s /etc/nginx/sites-avaliable/www.site.com /etc/nginx/sites-enable/www.site.com
#ln -s /etc/nginx/sites-avaliable/extranet.site.com /etc/nginx/sites-enable/extranet.site.com
#invoke-rc.d nginx restart
