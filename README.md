# docker-apache2-reverse-proxy
Dockerized apache2 reverse proxy service.

Get the content of ansible directory running:
```
https://github.com/jmferrer/ansible-apache2-reverse-proxy.git
```

Build image with:
```
docker build -t apache2-reverse-proxy:debian8 .
```

After creating virtualhosts in /some/place/config create docker container with:
```
docker create --name apache2-reverse-proxy --hostname apache2-reverse-proxy -p 443:443 -p 80:80 -v /some/place:/etc/apache2/sites-enabled apache2-reverse-proxy:debian8
```

## logs
**-v /some/place/logs:/var/log/apache2**

## ssl certificates
**-v /some/place/certs:/etc/apache2/certs**

## virtualhosts
**-v /some/place/config:/etc/apache2/sites-enabled**

### http proxy example
```
<VirtualHost *:80>
        ServerName example.com

        ProxyPass / http://example.com/
        ProxyPassReverse / http://example.com/

        ErrorLog /var/log/apache2/example.com-error.log
        CustomLog /var/log/apache2/example.com-access.log combined

</VirtualHost>
```
### ajp proxy example
```
<VirtualHost *:443>
    ServerName example.com

    ProxyPass / ajp://127.0.0.1:8009/
    ProxyPassReverse / ajp://127.0.0.1:8009/

    ErrorLog /var/log/apache2/example.com-error.log
    CustomLog /var/log/apache2/example.com-access.log combined
</VirtualHost>
```

### https proxy with http redirection example
```
<VirtualHost *:80>
    ServerName example.com

    # This will enable the Rewrite capabilities
    RewriteEngine On

    # This checks to make sure the connection is not already HTTPS
    RewriteCond %{HTTPS} !=on

    # This rule will redirect users from their original location, to the same location but using HTTPS.
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R,L]
</VirtualHost>
<VirtualHost *:443>
    ServerName example.com

    ProxyPass / ajp://127.0.0.1:8009/
    ProxyPassReverse / ajp://127.0.0.1:8009/

    SSLEngine on
    SSLProxyEngine on

    SSLCertificateFile /etc/apache2/certs/example.com.crt
    SSLCertificateKeyFile /etc/apache/certs/example.com.key
    SSLCertificateChainFile /etc/apache/certs/authority.crt

    ErrorLog /var/log/httpd/example.com-error.log
    CustomLog /var/log/httpd/example.com-access.log combined
</VirtualHost>
```

## ports
If you want the container listening in other ports than 80 and 443 (i.e. 8443) create a /some/place/config/ports.conf file with content:
```
Listen 8443
```
