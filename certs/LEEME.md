# Certificado autofirmado

El certificado público del sitio es:

```text
images.alumno.local.crt
```

Lo generé con:

```bash
mkdir -p /etc/ssl/dpl

openssl req -x509 -nodes -newkey rsa:2048 -days 365 \
    -keyout /etc/ssl/dpl/images.alumno.local.key \
    -out /etc/ssl/dpl/images.alumno.local.crt \
    -subj "/CN=images.alumno.local" \
    -addext "subjectAltName=DNS:images.alumno.local,DNS:www.images.alumno.local"

chmod 400 /etc/ssl/dpl/images.alumno.local.key
```

La clave privada queda en `/etc/ssl/dpl/` y no se incluye en el repositorio.
