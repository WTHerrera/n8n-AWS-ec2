## 🚀 Configuración de AWS para Docker🐳 y Nginx y certificado 🔒 con Certbot con n8n 🤖
> este procedimiento se actualizpo a partir de => https://github.com/Josh1313/n8n_AWS_installation

### Paso 0: Creación de una instancia en EC2 en AWS:
- Seguior el siguiente Tutorial => `Agregar tutorial`
- Despues de varios pasos debemos tener una instancia creada como: `i-05957bae28513bf8e (n8n-AWS)`
 
---
### Paso 1: Conectarse a la instancia EC2 de AWS mediante SSH
- Abrir terminal **como administrador**. En Windows  escribir `cmd` en **Buscar** (*esquina inferior iaquierdo, al lado del simbolo de Windows*), luego clik en  `Ejecutar como administrador`.
- Luego ir a la carpeta donde se descargó el archivo de EC2 (AWS), ej. "`n8npem.pem`", ej. `D:` luego `cd  00 Instalacion-NWN-AWS`, en el terminal deberia vizualisarse algo así `D:\00 Instalacion-N8N-AWS>` y iniciar con la configuración:  
```bash
ssh -i tuclave.pem ec2-user@<IP-publico>
```
- No te olvides de reemplazar `tuclave` con el nombre del archivo `n8nclave.pem` que creaste enEC2 cuando creaste **Par de claves** e `<IP-publico>` por el IP que se gerner´cuando creaste la instancia en EC2, ej. `5.145.30.95`, ej. ssh -i n8nclave.pem ec2-user@5.145.30.95

### Paso 2: Actualizar la instancia e instalar Docker
- Para actualizar los paquetes de tu sistema:
```bash
sudo yum update -y
```
- Para instalar Docker:
```bash
sudo yum install -y docker
```

### Paso 3: Iniciar y habilitar el servicio Docker
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### Paso 4: Añadir usuario al grupo Docker para acceso no root
```bash
sudo usermod -aG docker ec2-user
```

### Paso 5: Salir y volver a iniciar sesión para que los cambios surtan efecto.
```bash
exit
```

### Paso 6: Conectarse a la instancia AWS EC2 mediante SSH
```bash
ssh -i yourkey.pem ec2-user@publicip
```

### Paso 7: Instalar Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Paso 8: Ejecutar el contenedor Docker N8N
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
