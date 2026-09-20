# Despliegue en Oracle Cloud (Ubuntu) — Manual de Electricidad Automotriz

> Documento para la IA/persona que montará el sitio. Todo lo necesario está en esta misma carpeta.
> El sitio es **estático** (HTML + un PDF). No hay backend, ni base de datos, ni build. Solo servir archivos.

---

## 1. Qué es este proyecto

Una página web de una sola pantalla que:
- Muestra un **aviso** de que la guía fue hecha con IA y no revisada por un especialista.
- Ofrece un botón para **descargar el PDF** del manual.

Dominio deseado: **guia.capygamerstore.com**

## 2. Archivos de esta carpeta (lo que hay que subir)

```
index.html                          → la página (raíz del sitio)
manual-electricidad-automotriz.pdf  → el PDF que descarga el botón (~9 MB)
README.md                           → descripción corta
DESPLIEGUE-ORACLE.md                → este documento
```

Solo se sirven `index.html` y el `.pdf`. Los `.md` son documentación; no estorban pero puedes no subirlos.
El `index.html` referencia el PDF con la ruta relativa `manual-electricidad-automotriz.pdf`, así que **ambos deben quedar en la misma carpeta**.

## 3. Servidor objetivo

- Oracle Cloud Infrastructure (OCI), instancia **Ubuntu** (22.04 o 24.04 LTS).
- Servir con **Nginx** (recomendado) sobre HTTP/HTTPS.

## 4. Pasos de despliegue (copiar y pegar en la instancia Ubuntu)

### 4.1 Instalar Nginx
```bash
sudo apt update
sudo apt install -y nginx
```

### 4.2 Copiar los archivos del sitio
Sube `index.html` y `manual-electricidad-automotriz.pdf` a la instancia (por ejemplo con `scp` desde tu PC) y colócalos en el directorio web:
```bash
sudo mkdir -p /var/www/guia
sudo cp index.html manual-electricidad-automotriz.pdf /var/www/guia/
sudo chown -R www-data:www-data /var/www/guia
```

### 4.3 Configurar el sitio en Nginx
```bash
sudo tee /etc/nginx/sites-available/guia >/dev/null <<'NGINX'
server {
    listen 80;
    listen [::]:80;
    server_name guia.capygamerstore.com;

    root /var/www/guia;
    index index.html;

    # Descargas grandes y tipo correcto para el PDF
    location = /manual-electricidad-automotriz.pdf {
        default_type application/pdf;
        add_header Content-Disposition 'attachment; filename="Manual-Electricidad-Automotriz.pdf"';
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
NGINX

sudo ln -sf /etc/nginx/sites-available/guia /etc/nginx/sites-enabled/guia
sudo rm -f /etc/nginx/sites-enabled/default   # opcional: quita el sitio por defecto
sudo nginx -t && sudo systemctl reload nginx
```

### 4.4 Abrir el puerto en Oracle Cloud
Oracle bloquea puertos por defecto en DOS capas — hay que abrir ambas:

1. **Security List / NSG (en la consola de OCI):** en la subred de la instancia, añade reglas de entrada (Ingress):
   - Origen `0.0.0.0/0`, TCP puerto **80**
   - Origen `0.0.0.0/0`, TCP puerto **443**
2. **Firewall del sistema (iptables en Ubuntu de Oracle):**
```bash
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT
sudo netfilter-persistent save   # si no existe: sudo apt install -y iptables-persistent
```

En este punto, `http://<IP-PUBLICA-DE-LA-INSTANCIA>/` ya debe mostrar la página.

## 5. Dominio guia.capygamerstore.com (DNS)

En el panel DNS de **capygamerstore.com**, crea un registro:

| Tipo | Nombre / Host | Valor / Destino                    | TTL  |
|------|---------------|------------------------------------|------|
| A    | `guia`        | `<IP-PUBLICA-DE-LA-INSTANCIA-OCI>` | 3600 |

(Usa la IP pública reservada de la instancia OCI. Si prefieres CNAME, apunta a un hostname que resuelva a esa IP; para un subdominio con IP fija, el registro **A** es lo correcto.)

Comprueba con: `nslookup guia.capygamerstore.com` hasta que devuelva la IP.

## 6. HTTPS gratis (Let's Encrypt) — hazlo después de que el DNS resuelva
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d guia.capygamerstore.com --non-interactive --agree-tos -m capygamerstore@gmail.com --redirect
```
Certbot edita el Nginx, instala el certificado y fuerza HTTPS. Se renueva solo (`systemctl status certbot.timer`).

## 7. Actualizar el PDF o la página en el futuro
Solo reemplaza el archivo y recarga (no hace falta reiniciar nada para archivos estáticos):
```bash
sudo cp nuevo.pdf /var/www/guia/manual-electricidad-automotriz.pdf
# la página se actualiza igual copiando el index.html nuevo
```

## 8. Notas / decisiones tomadas
- Sitio 100 % estático: la forma más simple y barata; Nginx sirviendo archivos.
- El PDF pesa ~9 MB (comprimido desde 65 MB reduciendo la resolución de las imágenes). Si se quiere más liviano, recomprimir las imágenes del PDF.
- No hay analítica ni cookies ni formularios: nada de privacidad que gestionar.
- `<meta name="robots" content="noindex">` está puesto en el index para que no lo indexen los buscadores (es material del grupo del curso). Quítalo si se quiere que aparezca en Google.

## 9. Alternativa sin servidor (si no se quiere administrar la instancia)
El sitio también funciona tal cual en cualquier hosting estático (GitHub Pages, Cloudflare Pages, Netlify). De hecho ya está publicado temporalmente en:
`https://darkshadow3124.github.io/guia-electricidad-automotriz/`
