Open Data en la UGR
============

Repositorio para temas relacionados con el [servidor de datos abiertos de la UGR](http://opendata.ugr.es).

## Uso

1.- Pondremos la dirección IP de la máquina en la que vamos a realizar dicha instalación en el fichero aprovisionamiento/ansible_hosts.
```
[ckan]
192.168.33.33
```

2.- Realizaremos los cambios necesarios en el playbook de ansible aprovisionamiento/ckan.yml relativos al usuario de la máquina en la que vamos a instalar la aplicación y la base de datos.
```
...
remote_user: vagrant
vars:
  - db_name: ckan_default
  - db_user: ckan_default
  - db_password: vagrantpass
...
```

3.- Asegurarnos que tenemos acceso por ssh a la máquina.

4.- Ejecutamos el programa de aprovisionamiento.
```
./instalar.sh
```

## Nota

Este aprovisiomiento sólo está disponible para una instalación sobre Ubuntu 12.04.
Estamos trabajando para realizarlo sobre cualquier distribución Linux.

### URL

[opendata.ugr.es](opendata.ugr.es)
