
<img  align="left" width="150" style="float: left;" src="https://www.upm.es/sfs/Rectorado/Gabinete%20del%20Rector/Logos/UPM/CEI/LOGOTIPO%20leyenda%20color%20JPG%20p.png">
<img  align="right" width="60" style="float: right;" src="http://www.dit.upm.es/figures/logos/ditupm-big.gif">


<br/><br/>
# CDPS - Práctica 4: DOCKER

**Guía del Alumno**

## Descripción

En esta práctica el alumno ha de realizar una imagen Docker que permita arrancar un servidor web, similar a como ha realizado usando una máquina virtual pesada. Se indicarán diversas formas de tener una infraestructura Docker operativa, implementando buenas prácticas actuales.

## Objetivos de la Práctica

El objetivo de esta sesión no es solo ejecutar comandos, sino comprender el ecosistema de los contenedores modernos. Al finalizar esta práctica, el alumno será capaz de:

* **Comprender el paradigma de la contenerización:** Diferenciar claramente entre la virtualización tradicional (máquinas virtuales completas) y la virtualización ligera a nivel de sistema operativo (contenedores), asimilando sus ventajas en términos de rendimiento, consumo de recursos y portabilidad.
* **Manejar el ciclo de vida de los contenedores:** Dominar los comandos básicos del CLI de Docker para crear, ejecutar, inspeccionar, detener y eliminar tanto imágenes como contenedores.
* **Crear imágenes personalizadas (Mono-contenedor):** Aprender a escribir archivos `Dockerfile` eficientes para empaquetar aplicaciones (como un servidor web estático) junto con sus dependencias, asegurando que sean reproducibles en cualquier entorno.
* **Orquestar servicios complejos (Multi-contenedor):** Utilizar `Docker Compose` para definir, desplegar y conectar aplicaciones formadas por múltiples servicios (ej. un proxy inverso, una aplicación de backend y una base de datos) que interactúan a través de redes virtuales internas.
* **Gestionar la persistencia y seguridad:** Entender el uso de volúmenes de Docker para evitar la pérdida de datos y aplicar buenas prácticas de seguridad (como el uso de archivos `.env`) para evitar exponer credenciales en el código fuente.

## Actividades a desarrollar

Vamos a instalar y configurar un servidor web usando la tecnología de virtualización ligera Docker. Para ello veremos cómo replicar parte de la anterior práctica, pero utilizando una tecnología distinta.

Se entregará un archivo comprimido respondiendo a las cuestiones planteadas en la memoria y aportando los ficheros de configuración utilizados.

## Tarea 0: Instalación del entorno

Para la realización de esta práctica es necesario tener acceso a un entorno con Docker instalado. Tenemos diversas alternativas:

### Alternativa A: Entornos Cloud de Desarrollo (Recomendado)

Son soluciones modernas, estables y gratuitas que ofrecen un entorno completo de desarrollo en el navegador (basado en VS Code) con Docker ya preinstalado:

1. **GitHub Codespaces:** Accede a un repositorio de GitHub, pulsa en `<> Code` y crea un nuevo Codespace. Tendrás una terminal con Docker listo para usar.
2. **Gitpod:** Añade `gitpod.io/#` delante de la URL de cualquier repositorio público de GitHub para abrir un entorno virtualizado instantáneo.

### Alternativa B: Instancia en la nube "Play with Docker"

Es una solución muy sencilla pero limitada a sesiones de 4 horas y puede presentar problemas de carga en horas pico.

1. Accede a: <https://labs.play-with-docker.com>
2. Inicia sesión con Docker o GitHub.
3. Pulsa "Add new instance" para obtener una consola. **Nota:** Es necesario descargar todos los ficheros antes de cerrar la sesión, ya que se perderán al finalizar el tiempo.

### Alternativa C: Instalación en ordenador personal

Puedes instalar Docker en tu propia máquina:

* **Windows/Mac:** [Docker Desktop](https://docs.docker.com/desktop/) o alternativas más ligeras como OrbStack (en Mac).
* **Linux:** Instalación nativa mediante el gestor de paquetes de tu distribución (ej. `apt install docker.io docker-compose-v2`).

*Verifica que el entorno funciona ejecutando:*
```bash
docker run hello-world
```

> 📝 **Cuestión para la memoria (C1):** Indica qué solución de las anteriores has utilizado para realizar la práctica y justifica brevemente por qué.

## Tarea 1: Creación de imagen mono-contenedor

Ejecuta los siguientes comandos para familiarizarte con el ciclo de vida básico. 

```bash
docker run hello-world
docker ps -a
docker rmi -f hello-world
docker pull hello-world
```

> 📝 **Cuestión para la memoria (C2):** Explica detalladamente qué operación realiza cada uno de los cuatro comandos anteriores.
> 📝 **Cuestión conceptual (C3):** A raíz de lo observado, ¿cuál es la diferencia conceptual fundamental entre una "imagen" de Docker y un "contenedor" de Docker?

### Interactuando con un contenedor base

Ahora arrancaremos un contenedor de Debian y entraremos por una consola:

```bash
docker run -it debian
```

Úsalo como si fuese una máquina virtual. Modifica un fichero del sistema, sal del contenedor (escribiendo `exit`) y detén/elimina el contenedor. Vuelve a ejecutar el comando inicial para crear un contenedor nuevo.

> 📝 **Cuestión para la memoria (C4):** ¿Está el fichero que modificaste en este nuevo contenedor? Explica brevemente cómo gestiona Docker la persistencia de datos y la naturaleza efímera de los contenedores por defecto.

### Creando tu propia imagen de Nginx

Vamos a utilizar un servidor web usando una imagen existente y añadiendo nuestro propio contenido. Utilizaremos el editor de terminal `vi`.

*(Tip para `vi`: Presiona la tecla `i` para entrar en modo inserción y poder escribir. Cuando termines, presiona `Esc`, escribe `:wq` y pulsa `Enter` para guardar y salir).*

Crea un archivo `index.html`:
```bash
vi index.html
```

Crea un archivo `Dockerfile`:
```bash
vi Dockerfile
```

Contenido del `Dockerfile`:
```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html
```

Construye la imagen (¡No olvides el punto `.` al final, que indica el directorio actual!):
```bash
docker build -t my-nginx .
```

Ejecuta el contenedor en segundo plano (daemon) mapeando puertos:
```bash
docker run -d --name my-running-nginx -p 8080:80 my-nginx
```

Accede desde tu navegador a la IP de la máquina (o `localhost` si es en tu PC) por el puerto 8080.

> 📝 **Cuestión para la memoria (C5):** Compara la experiencia de arranque y consumo de recursos entre este contenedor Nginx que acabas de ejecutar y el arranque del servidor web en una máquina virtual completa como las usadas en prácticas anteriores.

Al acabar, elimina el contenedor:
```bash
docker rm -f my-running-nginx
```

## Tarea 2: Uso de Docker Compose (Node.js + Nginx)

Docker Compose nos permite orquestar múltiples contenedores de forma declarativa. Vamos a crear un proxy inverso donde Nginx atienda las peticiones y se las pase a una app Node.js.

Estructura de Archivos (crea los siguientes directorios y archivos):
```bash
mkdir docker-node-nginx
cd docker-node-nginx
```

**1. La aplicación Node.js (`server.js`)**
```javascript
const http = require('http');
const hostname = '0.0.0.0';
const port = 3000;

const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('¡Hola! Has pasado por Nginx y has llegado a Node.js\n');
});

server.listen(port, hostname, () => {
    console.log(`Servidor corriendo en el puerto ${port}`);
});
```

**2. El Dockerfile (Para Node)**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY server.js .
EXPOSE 3000
CMD ["node", "server.js"]
```

**3. Configuración de Nginx (`nginx.conf`)**
```nginx
events {}
http {
    server {
        listen 80;
        location / {
            # "app-node" es el nombre del servicio en docker-compose.yml
            proxy_pass http://app-node:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

**4. Orquestación con `docker-compose.yml`**
```yaml
services:
  app-node:
    build: .
    container_name: mi_backend_node
    restart: always

  web-nginx:
    image: nginx:alpine
    container_name: mi_proxy_nginx
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app-node
```

Levanta el sistema:
```bash
docker compose up -d
```

> 📝 **Cuestión conceptual (C6):** En el archivo `nginx.conf`, estamos apuntando a `http://app-node:3000`. Sin embargo, `app-node` no es una dirección IP válida en Internet. ¿Cómo es posible que Nginx logre comunicarse con el contenedor de Node.js utilizando ese nombre? ¿Qué mecanismo interno de Docker lo permite?

Puedes ver los registros de ambos servicios combinados con:
```bash
docker compose logs -f
```

Para detener y eliminar esta infraestructura temporal:
```bash
docker compose down
```

## Tarea 3: Orquestación avanzada (WordPress y MariaDB)

Ahora desplegaremos un entorno real de CMS. En lugar de poner las contraseñas en el archivo YAML en texto plano, usaremos variables de entorno, una práctica fundamental en DevOps.

**1. Crea un archivo oculto llamado `.env` en tu directorio de trabajo:**
```bash
vi .env
```

Contenido del `.env` (asegúrate de no subir este archivo a repositorios públicos reales):
```env
MYSQL_ROOT_PASSWORD=supersecreto_root
MYSQL_PASSWORD=secreto_wordpress
```

**2. Crea el archivo `docker-compose.yml`:**
```yaml
services:
  db:
    image: mariadb:lts
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=manager
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}

  web:
    image: wordpress:latest
    depends_on:
      - db
    volumes:
      - ./wp_data:/var/www/html
    environment:
      - WORDPRESS_DB_USER=manager
      - WORDPRESS_DB_PASSWORD=${MYSQL_PASSWORD}
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_NAME=wordpress
    ports:
      - "8080:80"

volumes:
  db_data:
```

Ejecuta el entorno con `docker compose up -d` y accede a `http://IP_DE_TU_MAQUINA:8080` para visualizar la pantalla de instalación de WordPress.

> 📝 **Cuestión conceptual (C7):** Observa la sección `volumes:` al final del documento YAML, donde se define `db_data`. Si omitiéramos la configuración de los volúmenes, ¿qué ocurriría con todos los artículos, páginas y configuraciones de nuestro WordPress si por algún motivo tuviéramos que ejecutar un `docker compose down` y luego volver a levantarlo?

## Entrega de la Práctica

La entrega final debe ser **un único archivo comprimido en formato `.zip`**. Este archivo debe contener los siguientes elementos:

1. **Memoria de la práctica (PDF):** Un documento en formato PDF que responda de forma clara y justificada a todas las cuestiones (C1 a C7) planteadas a lo largo de este enunciado, e incluya capturas de pantalla relevantes que demuestren el funcionamiento (ej. acceso a los servidores web por el puerto 8080).
2. **Archivos de configuración:**
   - El archivo `Dockerfile` y el `index.html` generados en la Tarea 1.
   - El código fuente (`server.js`), el archivo `nginx.conf`, el `Dockerfile` y el `docker-compose.yml` de la Tarea 2.
   - El archivo `docker-compose.yml` y un archivo de ejemplo `.env.example` (renombrando tu `.env` para la entrega) generados en la Tarea 3. 

*(Asegúrate de mantener las carpetas separadas para las Tareas 2 y 3 dentro del ZIP para facilitar su corrección).*
