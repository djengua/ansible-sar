
## Servidores
Ansible puede administrar solo servidores que son explicitamente conocidos. Puedes proveer información acerca de los servidores especificandole en un inventario. Usualmente deben estar dentro de una carpeta ```inventario```.

### Inventory QA example
```
[webservers]
testserver ansible_host=127.0.0.1

[webservers:vars]
ansible_user=usuario
ansible_password=somepassword # Para proporiconar el password debe tener la herramienta sshpass (instalación en mac => brew install sshpass)
```

### Ping
Para conectar al servidor llamado testserver (por ejemplo) descrito en el archivo inventario e invocar el modulo ping:
```bash
ansible testserver -i inventory/vagrant.ini -m ping
```
### Respuesta OK

```json
testserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/local/bin/python3.13"
    },
    "changed": false,
    "ping": "pong"
}
```

## Defaults
Con los defaults configurados, puedes invocar Ansible sin pasar el argumento -i, como sigue:
```bash
ansible testserver -m ping
```

## Uso de modulos de comando
Puedes utilizar cualquier modulo que quieras, por ejemplo, instalar nginx en un Ubuntu:

```bash
ansible testserver -b -m package -a name=nginx


# Playbooks
Scripts de administración de Ansible.

## Ejemplo de un playbook simple
Configurar un host para correr un servidor HTTP.

### Crear playbook.yml
Creando el ejmplo para levantar le servidor:

```yml

- name: Configure webserver with nginx
  hosts: webservers
  become: True
  tasks:
    - name: Ensure nginx is installed
      package: name=nginx update_cache=yes
    - name: Copy nginx config file
      copy:
        src: nginx.conf
      dest: /etc/nginx/sites-available/default
    - name: Enable configuration
      file: >
        dest=/etc/nginx/sites-enabled/default
        src=/etc/nginx/sites-available/default
        state=link
    - name: Copy index.html
      template: >
        src=index.html.j2
        dest=/usr/share/nginx/html/index.html
    - name: Restart nginx
      service: name=nginx state=restarted
```

Sobrescribimos la configuración default del nginx:

```
server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;
        root /usr/share/nginx/html;
        index index.html index.htm;
        server_name localhost;
        location / {
                try_files $uri $uri/ =404;
        }
}
```
El archivo va en la ruta ```playbooks/files/nginx.conf.```

Creamos una pagina web simple:

```
<html> <head>
    <title>Welcome to ansible</title>
  </head>
  <body>
  <h1>Nginx, configured by Ansible</h1>
  <p>If you can see this, Ansible successfully installed nginx.</p>
  <p>Running on {{ inventory_hostname }}</p>
  </body>
</html>
```
En la ruta ```playbooks/templates/index.html.j2.```



### Visualizar los grupos en el inventory
```bash
ansible-inventory --graph
```
Viendo algo como esto:
```
   @all:
      |--@ungrouped:
      |--@webservers:
      | |--testserver
```

## Correr el playbook
El comando ansible-playbook ejecuta los playbooks:
```bash
ansible-playbook webservers.yml
```
---

### Ejecutar playbook
```bash
ansible-playbook -i inventory/qa.ini playbooks/sar.yml
```