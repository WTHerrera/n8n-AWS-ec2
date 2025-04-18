## 🚀 Configuración de AWS para Docker🐳 y Nginx y certificado 🔒 con Certbot con n8n 🤖
> este procedimiento se actualizpo a partir de => https://github.com/Josh1313/n8n_AWS_installation

### 🟢 Paso 0: Creación de una instancia en EC2 en AWS:
- Seguior el siguiente Tutorial => `Agregar tutorial`
- Despues de varios pasos debemos tener una instancia creada como: `i-05957bae28513bf8e (n8n-AWS)`
 
---
### ✅ Paso 1: Conectarse a la instancia EC2 de AWS mediante SSH
- Abrir terminal **como administrador**. En Windows  escribir `cmd` en **Buscar** (*esquina inferior izquierdo, al lado del simbolo de Windows*), luego clik en  `Ejecutar como administrador`.
- Luego ir a la carpeta donde se descargó el archivo de EC2 (AWS), ej. "`n8npem.pem`", ej. `D:` luego `cd  00 Instalacion-NWN-AWS`, en el terminal deberia vizualisarse algo así `D:\00 Instalacion-N8N-AWS>` y iniciar con la configuración:  
```bash
ssh -i tuclave.pem ec2-user@<IP-publico>
```
- No te olvides de reemplazar `tuclave` con el nombre del archivo, ej. `n8nclave.pem`, que creaste en EC2 cuando creaste **Par de claves** e `<IP-publico>` por el IP que se gerneró cuando creaste la instancia en EC2, ej. `5.145.30.95`, ej. final `ssh -i n8nclave.pem ec2-user@5.145.30.95`

> ⚠ Tips Si tiene problemas con el copiar y pegar enel terminal, si no puedes pegar con **(Ctrl+V)**. Primero copia el tecto codigo con **Ctrl+C**(o como prefieras)  luego vas a la linea correspondiente del terminal y solo debes darle **clik secundario** y con este simple paso se pegará el texto o codigo copiado.


### ✅ Paso 2: Actualizar la instancia e instalar Docker en Amazon Linux 2023
- Para actualizar los paquetes de tu sistema, ejecutar:
```bash
sudo yum update -y
```
ó alternativamente ` sudo dnf update -y `

- Instalar los paquetes necesarios para Docker, ejecutar:  
```bash
sudo dnf install -y docker
```

> ⚠️ Si sale algun error 🚨, corregir usando comando `yum`  
```bash
sudo yum install -y docker
```

| Comando | ¿Qué es? | ¿Dónde se usa? |
|--|--|--|
| yum | El gestor de paquetes más antiguo | CentOS 7, RHEL 7, Amazon **Linux 2** |
| dnf | El reemplazo moderno de yum | Fedora, CentOS 8+, RHEL 8+, Amazon **Linux 2023** |

> Como nosotros hemos instalado **Linux 2023** lo correcto debe ser `dnf` 


### ✅ Paso 3: Iniciar y habilitar el servicio Docker
- Ejecutas:
```bash
sudo systemctl start docker
```
- Luego:
```bash
sudo systemctl enable docker
```

### ✅ Paso 4: Añadir usuario al grupo Docker para acceso no-root (*non-root access*)
```bash
sudo usermod -aG docker ec2-user
```

### ✅ Paso 5: Salir y volver a iniciar sesión para que los cambios surtan efecto.
```bash
exit
```

### ✅ Paso 6: Conectarse a la instancia AWS EC2 mediante SSH
```bash
ssh -i tuclave.pem ec2-user@<IP-publico>
```
> Y debe salir el mensaje: `ec2-user adm wheel systemd-journal docker`


### ✅ Paso 7: Instalar Docker Compose
7.1.  Corre esta línea para descargar Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
7.2. Corre esta línea para darle permisos de ejecución
```bash
sudo chmod +x /usr/local/bin/docker-compose
```


Aquí tienes una versión mejorada del texto, con mejor redacción, formato más claro y sugerencias para facilitar la comprensión:

---

### ✅ Paso 8: Ejecutar el contenedor Docker de N8N

> ⚠ **Requisito previo**: Debes tener un **subdominio configurado** (por ejemplo: `n8n.tudominio.com`) que apunte a la **IP pública** de tu instancia EC2.  
> Puedes usar tu propio dominio o servicios de terceros para ello.  
> 👉 Consulta el siguiente tutorial para configurar tu subdominio: `Agregar Tutorial`.

#### Comando para ejecutar N8N en Docker:
```bash
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="n8n.tudominio.com" \
-e WEBHOOK_TUNNEL_URL="https://n8n.tudominio.com/" \
-e WEBHOOK_URL="https://n8n.tudominio.com/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n
```


### Paso 8: Ejecutar el contenedor Docker N8N
> ⚠ Para ejecutar esta parte debes crear un subdominio en algun servidor propio o de terceros, ej. `n8n.tudominio.com`, que apunte a `<IP-Publico>` de EC2, para ello seguir el siguiente tutorial => `Agregar Tutorial`

```bash
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="your-domain-name" \
-e WEBHOOK_TUNNEL_URL="https://your-domain-name/" \
-e WEBHOOK_URL="https://your-domain-name/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n
```


### Paso 9: Instalar y configurar Nginx
```bash
sudo dnf install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Paso 10: Configurar Nginx para N8N Reverse Proxy
```bash
sudo nano /etc/nginx/conf.d/n8n.conf
```
Add the following content in `n8n.conf`:
```nginx
server {
    listen 80;
    server_name your-domain-name;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;

        # Headers for WebSocket support
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Upgrade $http_upgrade;

        # Additional headers for forwarding client info
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Save and exit using:
```bash
CTRL+O, ENTER, CTRL+X
```

### Paso 11: Probar la configuración de Nginx y reiniciar el servicio
```bash
sudo nginx -t
sudo systemctl restart nginx
```

### Paso 12: Configurar certificado SSL con Certbot### Paso 12: Configurar certificado SSL con Certbot
```bash
sudo dnf install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain-name
sudo systemctl restart nginx
```
