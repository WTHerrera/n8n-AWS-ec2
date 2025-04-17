#  :abacus: n8n en AWS - EC2 - Como instalar y configurar  - Tutorial en Español :robot:

A continuación, te presento un tutorial paso a paso para desplegar **n8n** en una instancia **AWS EC2** usando una máquina elegible para la capa gratuita (t3.micro). Las explicaciones están en español, y el contenido está optimizado para ser claro y conciso.

#### Requisitos Previos
- Una cuenta de **AWS** (regístrate en aws.amazon.com; elegible para capa gratuita).
- Conocimientos básicos de comandos SSH y terminal.
- Un dominio (opcional, para URL personalizada; puedes usar la IP pública de EC2 si no tienes uno).

---
## Paso 1: Lanzar una Instancia EC2
1.1. **Inicia Sesión en AWS Console**:
   - Accede al [AWS Management Console](https://aws.amazon.com/console/).
   - Ve a **EC2** > **Instancias** > **Lanzar Instancias**.

1.2. **Configura la Instancia**:
   - **Nombre**: Por ejemplo, "n8n-servidor".
   - **AMI**: Selecciona **Ubuntu Server 22.04 LTS** (elegible para capa gratuita).
   - **Tipo de Instancia**: Elige **t3.micro** (2 vCPU, 1 GB RAM, capa gratuita).
   - **Par de Claves**: Crea o selecciona un par de claves (e.g., `n8n-key.pem`) para acceso SSH. Guarda el archivo `.pem` de forma segura.
   - **Configuración de Red**:
     - Permite **HTTP** (puerto 80), **HTTPS** (puerto 443), y **SSH** (puerto 22) en el grupo de seguridad.
     - Agrega una regla TCP personalizada para el puerto predeterminado de n8n (**5678**) si planeas acceder directamente.
   - **Almacenamiento**: Usa 8 GB SSD por defecto (capa gratuita; aumenta a 10-20 GB si es necesario).

1.3. **Lanzar**: Haz clic en **Lanzar Instancia**. Espera a que la instancia esté **en ejecución** (verifica en la página de **Instancias**).


## Paso 2: Conectar a tu Instancia EC2
1. **Obtén la IP Pública**:
   - En la consola de EC2, selecciona tu instancia y anota la **Dirección IPv4 Pública** (e.g., `3.123.456.789`).

2. **Conéctate vía SSH**:
   - Abre una terminal (o usa PuTTY en Windows).
   - Ejecuta:
     ```bash
     chmod 400 n8n-key.pem
     ssh -i n8n-key.pem ubuntu@<IP_PÚBLICA>
     ```
     Reemplaza `<IP_PÚBLICA>` con la IP de tu instancia.
   - Confirma la conexión (escribe `yes` si se solicita).



## Paso 3: Instalar Docker
1. **Actualiza el Sistema**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Instala Docker**:
   ```bash
   sudo apt install docker.io -y
   sudo systemctl start docker
   sudo systemctl enable docker
   sudo usermod -aG docker ubuntu
   ```
   - Cierra la sesión y vuelve a conectarte por SSH para aplicar los cambios de grupo.

3. **Verifica Docker**:
   ```bash
   docker --version
   ```


## Paso 4: Desplegar n8n con Docker
1. **Ejecuta el Contenedor de n8n**:
   ```bash
   docker run -d --name n8n \
     -p 5678:5678 \
     -v n8n_data:/home/node/.n8n \
     --restart unless-stopped \
     docker.n8n.io/n8nio/n8n
   ```
   - `-d`: Ejecuta en modo desacoplado.
   - `-p 5678:5678`: Mapea el puerto de n8n.
   - `-v n8n_data`: Persiste los datos de n8n.
   - `--restart unless-stopped`: Reinicia automáticamente a menos que lo detengas manualmente.

2. **Verifica n8n**:
   ```bash
   docker ps
   ```
   - Asegúrate de que el contenedor de n8n esté en ejecución.

3. **Accede a n8n**:
   - En un navegador, ve a `http://<IP_PÚBLICA>:5678`.
   - Verás la pantalla de configuración de n8n. Crea un usuario administrador (e.g., correo/contraseña).
   - **Nota**: Esto usa HTTP y no es seguro para producción; consulta el Paso 6 para SSL.



## Paso 5: Configurar el Grupo de Seguridad
1. **Actualiza el Grupo de Seguridad**:
   - En la consola de AWS, ve a **EC2** > **Grupos de Seguridad** > Selecciona el grupo de tu instancia.
   - Asegúrate de que las reglas de entrada permitan:
     - SSH (puerto 22, origen: tu IP o 0.0.0.0/0 para flexibilidad).
     - HTTP (puerto 80, origen: 0.0.0.0/0).
     - HTTPS (puerto 443, origen: 0.0.0.0/0).
     - TCP Personalizado (puerto 5678, origen: 0.0.0.0/0; restrínjelo a tu IP para mayor seguridad).



## Paso 6: (Opcional) Configurar SSL y Proxy Inverso con Nginx
Para producción, asegura n8n con HTTPS usando Nginx y Let’s Encrypt.

1. **Instala Nginx**:
   ```bash
   sudo apt install nginx -y
   ```

2. **Instala Certbot** (para Let’s Encrypt):
   ```bash
   sudo snap install core; sudo snap refresh core
   sudo snap install --classic certbot
   sudo ln -s /snap/bin/certbot /usr/bin/certbot
   ```

3. **Configura Nginx**:
   - Crea un archivo de configuración para Nginx:
     ```bash
     sudo nano /etc/nginx/sites-available/n8n
     ```
   - Agrega:
     ```nginx
     server {
         listen 80;
         server_name <TU_DOMINIO_O_IP>;

         location / {
             proxy_pass http://localhost:5678;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
         }
     }
     ```
     - Reemplaza `<TU_DOMINIO_O_IP>` con tu dominio (si tienes uno) o la IP pública.
   - Habilita la configuración:
     ```bash
     sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
     sudo nginx -t
     sudo systemctl reload nginx
     ```

4. **Obtén un Certificado SSL**:
   ```bash
   sudo certbot --nginx -d <TU_DOMINIO>
   ```
   - Sigue las instrucciones para configurar HTTPS. Si usas solo la IP, omite este paso o usa un dominio.

5. **Accede a n8n de Forma Segura**:
   - Ve a `https://<TU_DOMINIO_O_IP>`. n8n debería cargarse de forma segura.



## Paso 7: Persistir Datos y Monitorear
1. **Verifica la Persistencia de Datos**:
   - Los datos de n8n se almacenan en el volumen `n8n_data`. Verifica:
     ```bash
     docker volume inspect n8n_data
     ```

2. **Monitorea la Instancia**:
   - Usa AWS CloudWatch (opcional) o `htop` para monitorear CPU/RAM.
   - Ejecuta `docker logs n8n` para solucionar problemas.

3. **Respalda los Datos**:
   - Haz copias de seguridad periódicas del directorio `~/.n8n`:
     ```bash
     sudo tar -czvf n8n_backup.tar.gz /home/ubuntu/.n8n
     ```



## Paso 8: Probar n8n para Agentes de IA
1. **Crea un Flujo de Trabajo**:
   - En n8n, crea un nuevo flujo de trabajo.
   - Agrega nodos como **HTTP Request** o **Code** para integrarte con APIs de IA (e.g., OpenAI, Hugging Face).
   - Prueba con una automatización simple (e.g., generar texto con IA).

2. **Optimiza**:
   - Si la CPU/RAM es insuficiente (e.g., para tareas de IA pesadas), actualiza a **t3.small** (~$20/mes) o agrega memoria swap:
     ```bash
     sudo fallocate -l 2G /swapfile
     sudo chmod 600 /swapfile
     sudo mkswap /swapfile
     sudo swapon /swapfile
     echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
     ```



# Resumen Estilo Infografía (Texto)

```
╔════════════════════════════════════════════════════╗
║   Cómo Configurar n8n en AWS EC2 (Paso a Paso)      ║
╚════════════════════════════════════════════════════╝

┌─[1. Lanzar Instancia EC2]──────────────────────────┐
│ 🔹 Consola AWS > EC2 > Lanzar Instancias           │
│ 🔹 Nombre: n8n-servidor                           │
│ 🔹 AMI: Ubuntu 22.04 LTS                          │
│ 🔹 Tipo: t3.micro (Capa Gratuita)                 │
│ 🔹 Clave: Crear n8n-key.pem                       │
│ 🔹 Seguridad: Permitir SSH(22), HTTP(80), HTTPS(443), TCP(5678) │
│ 🔹 Almacenamiento: 8-20 GB SSD                    │
└───────────────────────────────────────────────────┘

┌─[2. Conectar a Instancia]──────────────────────────┐
│ 🔹 SSH: ssh -i n8n-key.pem ubuntu@<IP_PÚBLICA>     │
│ 🔹 Verificar conexión                             │
└───────────────────────────────────────────────────┘

┌─[3. Instalar Docker]───────────────────────────────┐
│ 🔹 Actualizar: sudo apt update && upgrade          │
│ 🔹 Instalar: sudo apt install docker.io            │
│ 🔹 Habilitar: sudo systemctl enable docker         │
│ 🔹 Agregar usuario: sudo usermod -aG docker ubuntu │
└───────────────────────────────────────────────────┘

┌─[4. Desplegar n8n]─────────────────────────────────┐
│ 🔹 Ejecutar: docker run -d --name n8n -p 5678:5678 ... │
│ 🔹 Acceder: http://<IP_PÚBLICA>:5678              │
│ 🔹 Configurar usuario administrador               │
└───────────────────────────────────────────────────┘

┌─[5. Asegurar con Nginx + SSL]──────────────────────┐
│ 🔹 Instalar Nginx: sudo apt install nginx          │
│ 🔹 Instalar Certbot: sudo snap install certbot     │
│ 🔹 Configurar Nginx: proxy a localhost:5678        │
│ 🔹 Obtener SSL: sudo certbot --nginx               │
│ 🔹 Acceder: https://<DOMINIO_O_IP>                 │
└───────────────────────────────────────────────────┘

┌─[6. Monitorear y Respaldar]────────────────────────┐
│ 🔹 Verificar: docker logs n8n, htop                │
│ 🔹 Respaldar: tar -czvf n8n_backup.tar.gz ~/.n8n   │
└───────────────────────────────────────────────────┘

┌─[7. Probar para Agentes IA]────────────────────────┐
│ 🔹 Crear flujo en n8n                             │
│ 🔹 Agregar nodos IA (e.g., HTTP Request)          │
│ 🔹 Optimizar: Agregar swap o actualizar a t3.small │
└───────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════╗
║ 🚀 ¡n8n está listo! Automatiza tus agentes IA! 🚀  ║
╚════════════════════════════════════════════════════╝
```


---
## Notas Adicionales
- **Capa Gratuita de AWS**: La instancia t3.micro ofrece 750 horas/mes (suficiente para 1 instancia 24/7). Monitorea el uso para evitar costos (~$8-10/mes tras la capa gratuita).
- **Seguridad**: Limita el puerto 5678 a tu IP en producción. Siempre usa HTTPS para accesos externos.
- **Escalabilidad**: Si los agentes IA requieren más recursos, considera actualizar a **t3.small** o explorar AWS ECS para contenedores.
- **Solución de Problemas**: Si n8n no carga, verifica Docker (`docker ps`), reglas de seguridad, y logs de Nginx (`sudo journalctl -u nginx`).

¿Necesitas que detalle algún paso en particular o quieres que genere un flujo de trabajo de ejemplo para agentes IA en n8n? Si prefieres, puedo ayudarte a formatear el contenido en una herramienta específica para PDF.
