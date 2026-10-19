# AE3 — Galería con procesado de imágenes, módulos y HTTPS

## 1. Instalación

El módulo `image_filter`, ImageMagick y GoAccess los instalé anteriormente en `dpl-lab` durante la práctica guiada de módulos de nginx.

Para instalar y activar `image_filter` utilicé:

```bash
docker exec dpl-lab apt-get update
docker exec dpl-lab apt-get install -y libnginx-mod-http-image-filter
docker exec dpl-lab ls /etc/nginx/modules-enabled/
docker exec dpl-lab grep -R "load_module" /etc/nginx/modules-enabled/
docker exec dpl-lab nginx -t
```

También instalé ImageMagick para generar y comprobar las imágenes y GoAccess para analizar los logs.

Al comenzar esta práctica comprobé que seguían disponibles:

```bash
nginx -V
convert -version
goaccess --version
```

Obtuve estas versiones:

```text
nginx/1.24.0 (Ubuntu)
ImageMagick 6.9.12-98
GoAccess 1.8.1
```

En la salida de `nginx -V` también apareció:

```text
--with-http_image_filter_module=dynamic
```

Después comprobé que nginx cargaba el módulo:

```bash
cat /etc/nginx/modules-enabled/50-mod-http-image-filter.conf
```

La salida mostró:

```text
load_module modules/ngx_http_image_filter_module.so;
```

Por tanto, comprobé que `image_filter` seguía instalado y activo.

## 2. Generación de las imágenes

Las 20 imágenes de prueba las generé dentro de `dpl-lab` con ImageMagick.

Primero comprobé que ImageMagick seguía instalado y después creé la carpeta donde se guardan las imágenes:

```bash
docker exec dpl-lab apt-get install -y imagemagick
docker exec dpl-lab mkdir -p /var/www/images.alumno.local/img
```

Las generé como se indica en la práctica de image_filter:

```bash
docker exec dpl-lab bash -c 'for i in $(seq -w 1 20); do \
  convert -size 400x400 gradient:"#2f6fed"-"#8bc34a" \
    -fill white -pointsize 64 -gravity center -annotate 0 "IMG $i" \
    /var/www/images.alumno.local/img/image$i.jpg; done'
```

Y para comprobarlo:

```bash
docker exec dpl-lab identify /var/www/images.alumno.local/img/image01.jpg
```

Obtuve la salida:

```text
/var/www/images.alumno.local/img/image01.jpg JPEG 400x400 400x400+0+0 8-bit sRGB 12679B 0.000u 0:00.000
```

Luego funcionó correctamente.

### Problema encontrado

Al ejecutar `nginx -T` desde dentro del contenedor como usuario `alumno`, nginx no pudo leer la clave privada del certificado:

```text
cannot load certificate key "/etc/ssl/dpl/images.alumno.local.key":
BIO_new_file() failed
Permission denied
```

La clave privada tenía permisos restringidos y el usuario `alumno` no podía leerla. En lugar de cambiarlos, salí del contenedor y repetí la comprobación desde el anfitrión, ejecutando nginx como `root` dentro de `dpl-lab`:

```bash
docker exec dpl-lab nginx -T
```

Aquí sí funcionó la comprobación.
