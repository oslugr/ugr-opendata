# <center>Instalación de CKAN en Debian Jessie</center>

`su`

`apt-get update && apt-get dist-upgrade`

`apt-get install python-dev postgresql libpq-dev python-pip python-virtualenv git-core solr-jetty8 openjdk-7-jdk python-pastescript apache2 libapache2-mod-wsgi libapache2-mod-rpaf nginx build-essential libxslt1-dev libxml2-dev git`

`apt-get install postfix` 

[Internet site](pruebackan1404.pruebackan1404.a10.internal.cloudapp.net)

`mkdir -p /usr/lib/ckan/default` 

chown \`whoami` /usr/lib/ckan/default

`virtualenv --no-site-packages /usr/lib/ckan/default`

`. /usr/lib/ckan/default/bin/activate`

`pip install -e 'git+https://github.com/ckan/ckan.git@ckan-2.3#egg=ckan'`

`pip install -r /usr/lib/ckan/default/src/ckan/requirements.txt`

`deactivate`              

 `/usr/lib/ckan/default/bin/activate`

`-u postgres createuser -S -D -R -P ckan_default`

`-u postgres createdb -O ckan_default ckan_default -E utf-8`

`mkdir -p /etc/ckan/default`

chown -R \`whoami` /etc/ckan/

`cd /usr/lib/ckan/default/src/ckan`

`paster make-config ckan /etc/ckan/default/development.ini`

`nano /etc/ckan/default/development.ini`


+ Cambiar en **pass** por contraseña del usuario *ckan_default*:

sqlalchemy.url = postgresql://ckan_default:**pass**@localhost/ckan_default

+ Comprobar -> **ckan.site_id = default**

`nano /etc/default/jetty8`

**NO_START=0**

**JETTY_HOST=127.0.0.1**

**JETTY_PORT=8983**


`service jetty8 start`

`mv /etc/solr/conf/schema.xml /etc/solr/conf/schema.xml.bak`

`ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml /etc/solr/conf/schema.xml`

`service jetty8 restart`


`curl http://localhost:8983/solr/`

`nano /etc/ckan/default/development.ini`

`solr_url=http://127.0.0.1:8983/solr`

`cd /usr/lib/ckan/default/src/ckan`

`paster db init -c /etc/ckan/default/development.ini`


`ln -s /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/who.ini`


+ (Asegurando de que **. /usr/lib/ckan/default/bin/activate)**

`cd /usr/lib/ckan/default/src/ckan`

`paster serve /etc/ckan/default/development.in`

`curl http://pruebackan1404.cloudapp.net:5000/`

`cp /etc/ckan/default/development.ini /etc/ckan/default/production.ini`



### <center>DESPLEGAR INSTALACIÓN PARA PRODUCCIÓN</center>



`nano /etc/apache2/apache2.conf`

> añadir -> **ServerName localhost** al final del archivo


`nano /etc/ckan/default/apache.wsgi`




```
import os
activate_this = os.path.join('/usr/lib/ckan/default/bin/activate_this.py')
execfile(activate_this, dict(__file__=activate_this))

from paste.deploy import loadapp
config_filepath = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'production.ini')
from paste.script.util.logging_config import fileConfig
fileConfig(config_filepath)
application = loadapp('config:%s' % config_filepath)
```


`nano /etc/apache2/sites-available/ckan_default.conf`


```
WSGISocketPrefix /var/run/wsgi
<VirtualHost 127.0.0.1:8080>
    ServerName default.pruebackan1404.cloudapp.net
    ServerAlias www.default.pruebackan1404.cloudapp.net
    WSGIScriptAlias / /etc/ckan/default/apache.wsgi

    # Pass authorization info on (needed for rest api).
    WSGIPassAuthorization On

    # Deploy as a daemon (avoids conflicts between CKAN instances).
    WSGIDaemonProcess ckan_default display-name=ckan_default processes=2 threads=15

    WSGIProcessGroup ckan_default
    
    # Allow access to all connections
    <Directory /etc/ckan/default>
        Options All
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/apache2/ckan_default.error.log
    CustomLog /var/log/apache2/ckan_default.custom.log combined

    <IfModule mod_rpaf.c>
        RPAFenable On
        RPAFsethostname On
        RPAFproxy_ips 127.0.0.1
    </IfModule>
</VirtualHost>
```

`nano /etc/apache2/ports.conf`

```
Cambiar "Listen 80" por "Listen 8080"
```
`nano /etc/nginx/sites-available/ckan_default`


```
proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=cache:30m max_size=250m;
proxy_temp_path /tmp/nginx_proxy 1 2;

server {
    client_max_body_size 100M;
    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $host;
        proxy_cache cache;
        proxy_cache_bypass $cookie_auth_tkt;
        proxy_no_cache $cookie_auth_tkt;
        proxy_cache_valid 30m;
        proxy_cache_key $host$scheme$proxy_host$request_uri;
        # In emergency comment out line to force caching
        # proxy_ignore_headers X-Accel-Expires Expires Cache-Control;
    }

}
```

`chmod -R 775 /etc/ckan`

`a2ensite ckan_default`

`a2dissite 000-default`

`rm -vi /etc/nginx/sites-enabled/default`

`ln -s /etc/nginx/sites-available/ckan_default /etc/nginx/sites-enabled/ckan_default`

`service apache2 reload`

`service nginx reload`

**Aquí no me iniciaba nginx con apache2, (is set to 1 on the firewall computer):** [Fuente](http://www.linuxquestions.org/questions/linux-software-2/unable-to-load-images-4175514033/)

`nano /proc/sys/net/ipv4/ip_forward`

### INSTALAR FILESTORE

`mkdir -p /var/lib/ckan/default`

`nano /etc/ckan/default/production.ini`

>  Descomentar
`ckan.storage_path = /var/lib/ckan`
>  Cambiar a 
`ckan.storage_path = /var/lib/ckan/default`
```
chown www-data /var/lib/ckan/default
chmod u+rwx /var/lib/ckan/default
```

### INSTALAR DATASTORE

`nano /etc/ckan/default/production.ini`
> añadir 
`datastore` 
> a la lista 
`ckan.plugins`

`exit` para salir de **root**, pues si no, no me funcionaba:

`sudo -u postgres createuser -S -D -R -P -l datastore_default`

`sudo -u postgres createdb -O ckan_default datastore_default -E utf-8`

> Volvemos a **root**:

`sudo su`

`nano /etc/ckan/default/production.ini`
> Descomentar y adaptar `pass`

`ckan.datastore.write_url = postgresql://ckan_default:prueba@localhost/datastore_default`

`ckan.datastore.read_url = postgresql://datastore_default:prueba@localhost/datastore_default`


`cd /etc/ckan/default/`

`paster --plugin=ckan datastore set-permissions -c production.ini | sudo -u postgres psql`

> Si da error es por `sudo`, hacemos lo mismo que antes

`curl http://pruebackan1404.cloudapp.net/api/3/action/datastore_search?resource_id=_table_metadata`

`curl -X GET "http://localhost/api/3/action/datastore_search?resource_id=_table_metadata"`


### INSTALANDO DATAPUSHER

`virtualenv /usr/lib/ckan/default/datapusher`

`mkdir /usr/lib/ckan/default/datapusher/src`

`cd /usr/lib/ckan/default/datapusher/src`

`git clone -b stable https://github.com/ckan/datapusher.git`

`cd datapusher`

`/usr/lib/ckan/default/datapusher/bin/pip install -r requirements.txt`

`/usr/lib/ckan/default/datapusher/bin/python setup.py develop`

`nano /etc/apache2/sites-available/datapusher.conf`

```
WSGISocketPrefix /var/run/wsgi
<VirtualHost 127.0.0.1:8800>
    ServerName datapusher.pruebackan1404.cloudapp.net
    ServerAlias www.datapusher.pruebackan1404.cloudapp.net
    WSGIScriptAlias / /etc/ckan/default/datapusher.wsgi

    # Pass authorization info on (needed for rest api)
    WSGIPassAuthorization On

    # Deploy as a daemon (avoids conflicts between CKAN instances)
    WSGIDaemonProcess datapusher display-name=demo processes=1 threads=15

    WSGIProcessGroup datapusher

    # Allow access to all connections
    <Directory /etc/ckan/default>
        Options All
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/apache2/datapusher.error.log
    CustomLog /var/log/apache2/datapusher.custom.log combined

</VirtualHost>
```

`cp deployment/datapusher.wsgi /etc/ckan/default`

`cp deployment/datapusher_settings.py /etc/ckan/default`

`nano /etc/apache2/ports.conf`
> Añadir 
`"Listen 8800"`

`a2ensite datapusher`
`service apache2 reload`

`nano /etc/ckan/default/production.ini`
> Descomentar y cambiar
`ckan.datapusher.url = http://0.0.0.0:8800/`
> Buscar y cambiar
`ckan.site_url = http://pruebackan1404.cloudapp.net/`
> Buscar y añadir 
`ckan.plugins = <other plugins> datapusher`

`service apache2 restart`

`chmod -R 775 /etc/ckan`


### Instalar PDF_VIEW

> Activate:
`. /usr/lib/ckan/default/bin/activate`
`pip install ckanext-pdfview`
`nano /etc/ckan/default/production.ini`

> Añadir `pdf_view` a `ckan.plugins` y `ckan.views.default_views`
> Añadir `pdf` a `ckan.preview.loadable`
> Añadir `resource_proxy` a `ckan.plugins`

`service apache2 reload`


**Configuraciones:**

[ckan](http://ckan.readthedocs.org/en/latest/maintaining/getting-started.html)
[ckan1](http://ckan.readthedocs.org/en/latest/sysadmin-guide.html)
[ckan2](http://ckan.readthedocs.org/en/latest/maintaining/index.html)
[ckan3](http://docs.ckan.org/en/latest/maintaining/data-viewer.html)
[ckan4](http://ckan.readthedocs.org/en/latest/theming/index.html)


Por [Germán](germaaan@gmail.com)
