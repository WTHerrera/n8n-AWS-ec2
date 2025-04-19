#  :abacus: Como configurar una instancia en EC2(AWS) para instalar n8n :robot:

A continuación, te presento un tutorial paso a paso para desplegar **n8n** en una instancia **AWS EC2** usando una máquina elegible para la capa gratuita (t3.micro). Las explicaciones están en español, y el contenido está optimizado para ser claro y conciso.

#### Requisitos Previos
- Una cuenta de **AWS** (regístrate en aws.amazon.com; elegible para capa gratuita).
- Conocimientos básicos de comandos SSH y terminal.
- Un dominio (opcional, para URL personalizada; puedes usar la IP pública de EC2 si no tienes uno).

---
## ✅ Paso 1: Lanzar una Instancia EC2
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


## ✅ Paso 2: Conectar a tu Instancia EC2
1. **Obtén la IP Pública**:
   - En la consola de EC2, selecciona tu instancia y anota la **Dirección IPv4 Pública** (e.g., `3.123.456.789`).

2. **Conéctate vía SSH**:

> Sigue los pasos en el **tutorial** => `Agregar tutorial`




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
