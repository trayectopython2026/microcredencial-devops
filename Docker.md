# 🐳 Fundamentos de DevOps — Docker + Cloudflare Tunnel

En esta práctica vamos a crear una página web sencilla, ejecutarla dentro de un contenedor Docker utilizando **Nginx** y finalmente publicarla temporalmente en Internet utilizando **Cloudflare Tunnel**.

---

# 1. Instalar Docker

Primero actualizamos los repositorios:

```bash
sudo apt update
```

Instalamos Docker:

```bash
sudo apt install docker.io -y
```

---

# 2. Inicializar Docker

Habilitamos Docker para que se inicie automáticamente junto con el sistema:

```bash
sudo systemctl enable --now docker
```

Podemos comprobar que está funcionando con:

```bash
sudo systemctl status docker
```

Para salir de esta pantalla presionamos:

```text
q
```

---

# 3. Comprobar la versión de Docker

```bash
docker --version
```

Deberíamos ver algo similar a:

```text
Docker version 28.x.x
```

---

# 4. Probar Docker

Ejecutamos el contenedor de prueba:

```bash
sudo docker run hello-world
```

Si Docker está funcionando correctamente aparecerá:

```text
Hello from Docker!
```

✅ Docker está instalado correctamente.

---

# 5. Ver contenedores funcionando

Para ver los contenedores que están actualmente en ejecución:

```bash
sudo docker ps
```

Para ver también los contenedores detenidos:

```bash
sudo docker ps -a
```

---

# 6. Crear nuestro proyecto

Creamos una carpeta para nuestra práctica:

```bash
mkdir practica-devops
```

Entramos a la carpeta:

```bash
cd practica-devops
```

---

# 7. Crear nuestra página web

Creamos el archivo:

```bash
nano index.html
```

Agregamos el siguiente código:

```html
<!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8">
    <title>Fundamentos de DevOps</title>
</head>

<body>

    <h1>Fundamentos de DevOps</h1>

    <h2>Juan Pérez</h2>

    <p>Mi primera aplicación Docker</p>

</body>

</html>
```

Guardamos con:

```text
CTRL + O
```

Presionamos:

```text
ENTER
```

Y salimos de Nano con:

```text
CTRL + X
```

---

# 8. Crear el Dockerfile

Dentro de la misma carpeta creamos:

```bash
nano Dockerfile
```

⚠️ El archivo debe llamarse exactamente:

```text
Dockerfile
```

Sin extensión.

Dentro colocamos:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

Guardamos y salimos.

---

# 9. Construir nuestra imagen Docker

Ejecutamos:

```bash
sudo docker build -t web-devops .
```

### ¿Qué significa?

- `docker build` → construye una imagen.
- `-t web-devops` → le asigna el nombre `web-devops`.
- `.` → indica que el Dockerfile está en la carpeta actual.

Podemos ver nuestras imágenes con:

```bash
sudo docker images
```

---

# 10. Ejecutar nuestro contenedor

Ejecutamos:

```bash
sudo docker run -d --name web-devops-container -p 8080:80 web-devops
```

### ¿Qué significa?

- `-d` → ejecuta el contenedor en segundo plano.
- `--name web-devops-container` → asigna un nombre al contenedor.
- `-p 8080:80` → conecta el puerto `8080` de nuestra computadora con el puerto `80` del contenedor.
- `web-devops` → es la imagen que queremos ejecutar.

---

# 11. Verificar el contenedor

Ejecutamos:

```bash
sudo docker ps
```

Deberíamos ver nuestro contenedor:

```text
web-devops-container
```

---

# 12. Abrir nuestra aplicación

Abrimos el navegador y escribimos:

```text
http://localhost:8080
```

También podemos comprobarlo desde la terminal:

```bash
curl http://localhost:8080
```

🎉 Nuestra primera aplicación está funcionando dentro de Docker.

---

# 🌎 Publicar nuestra aplicación con Cloudflare Tunnel

Hasta ahora nuestra web solamente puede verse desde nuestra computadora.

Ahora vamos a generar una dirección pública temporal para compartirla por Internet.

---

# 13. Preparar instalación de Cloudflared

Creamos la carpeta para las claves:

```bash
sudo mkdir -p --mode=0755 /usr/share/keyrings
```

Descargamos la clave oficial de Cloudflare:

```bash
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null
```

---

# 14. Agregar el repositorio de Cloudflare

Ejecutamos:

```bash
echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared any main" | sudo tee /etc/apt/sources.list.d/cloudflared.list
```

Actualizamos los repositorios:

```bash
sudo apt update
```

Instalamos Cloudflared:

```bash
sudo apt install cloudflared -y
```

---

# 15. Comprobar Cloudflared

```bash
cloudflared --version
```

---

# 16. Crear nuestro túnel

Primero comprobamos que nuestra aplicación sigue funcionacode .ndo:

```bash
curl http://localhost:8080
```

Luego ejecutamos:

```bash
cloudflared tunnel --url http://localhost:8080
```

Después de unos segundos Cloudflare nos mostrará una dirección parecida a:

```text
https://ejemplo-palabras.trycloudflare.com
```

Podemos copiar esa dirección y abrirla desde:

- Una computadora.
- Un celular.
- Otra conexión a Internet.

🌎 Nuestra aplicación Docker ahora puede verse desde Internet.

> ⚠️ La dirección generada por este tipo de túnel es temporal.
> Si cerramos Cloudflared y volvemos a ejecutarlo, probablemente obtendremos otra URL.

---

# 🔄 Modificar nuestra aplicación

Ahora vamos a modificar nuestra página.

Abrimos:

```bash
nano index.html
```

Y reemplazamos el contenido por:

```html
<!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8">
    <title>Fundamentos de DevOps</title>
</head>

<body>

    <h1>Fundamentos de DevOps</h1>

    <h2>Mi primera aplicación con Docker</h2>

    <p>Curso de DevOps</p>

</body>

</html>
```

Guardamos los cambios.

---

# 17. Detener el contenedor anterior

Como nuestro contenedor anterior sigue utilizando el puerto `8080`, primero debemos detenerlo:

```bash
sudo docker stop web-devops-container
```

---

# 18. Eliminar el contenedor anterior

```bash
sudo docker rm web-devops-container
```

Esto elimina el **contenedor**, pero nuestra imagen sigue existiendo.

---

# 19. Construir nuevamente la imagen

Como modificamos `index.html`, debemos reconstruir nuestra imagen:

```bash
sudo docker build -t web-devops .
```

---

# 20. Crear nuevamente el contenedor

Ejecutamos:

```bash
sudo docker run -d --name web-devops-container -p 8080:80 web-devops
```

Comprobamos:

```bash
sudo docker ps
```

Y nuevamente visitamos:

```text
http://localhost:8080
```

Ahora deberíamos ver los cambios realizados.

---

# 🔄 Flujo de trabajo al modificar nuestro proyecto

Cada vez que modificamos nuestro `index.html` podemos realizar:

```bash
sudo docker stop web-devops-container
```

```bash
sudo docker rm web-devops-container
```

```bash
sudo docker build -t web-devops .
```

```bash
sudo docker run -d --name web-devops-container -p 8080:80 web-devops
```

Podemos comprobarlo con:

```bash
curl http://localhost:8080
```

---

# 📁 Estructura final del proyecto

Nuestro proyecto debería tener solamente estos archivos:

```text
practica-devops/
│
├── Dockerfile
└── index.html
```

---

# 🧠 Conceptos aprendidos

Durante esta práctica utilizamos:

- Docker
- Imágenes
- Contenedores
- Dockerfile
- Nginx
- Puertos
- `docker build`
- `docker run`
- `docker ps`
- `docker stop`
- `docker rm`
- Cloudflare Tunnel

---

# 🚀 Flujo DevOps realizado

Podemos representar nuestra práctica de esta manera:

```text
Código HTML
     ↓
Dockerfile
     ↓
Construcción de imagen
     ↓
Imagen Docker
     ↓
Contenedor
     ↓
Nginx
     ↓
localhost:8080
     ↓
Cloudflare Tunnel
     ↓
Internet
```

---

# 🎯 Desafío para los alumnos

Modificar la aplicación agregando:

1. Nombre y apellido.
2. Nombre del curso.
3. Un título diferente.
4. Una breve descripción personal.
5. Una lista con tres tecnologías que quieran aprender.

Por ejemplo:

```html
<h1>Mi primera aplicación DevOps</h1>

<h2>Juan Pérez</h2>

<p>Estoy aprendiendo Docker y DevOps.</p>

<h3>Tecnologías que quiero aprender</h3>

<ul>
    <li>Docker</li>
    <li>Linux</li>
    <li>AWS</li>
</ul>
```

Luego deberán:

```text
Modificar código
      ↓
Construir imagen
      ↓
Crear contenedor
      ↓
Probar en localhost
      ↓
Crear Cloudflare Tunnel
      ↓
Compartir la URL
```

---

## ✅ Objetivo cumplido

Al finalizar la práctica tendremos una aplicación web:

**HTML → Nginx → Docker → Cloudflare Tunnel → Internet**

🎉 ¡Nuestra primera aplicación web dockerizada y publicada!





