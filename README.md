# Servicios de extensión de chromium

Esto es un servicio usado por [Notificador Chromium](https://github.com/PinkLittleKitty/chromium-notifier) para tener un seguimiento de errores y para aumentar la privacidad al buscar información de actualizaciones para las extensiones instaladas desde la Chrome Web Store (extrae cookies con datos personales).

## Requerimientos

- [Vercel](https://vercel.com/)
- Una base de datos MongoDB (para persistencia y caché)
- La variable de entorno `MONGODB_URI` proporcionada por Vercel Secrets (prod) o un archivo `.env` local (dev)

## Uso

Nota: Las respuestas que ves acá son todo lo que se guardó alguna vez, nada más - particularmente datos de usuario - se recolecta.

### Seguimiento de errores

Ayuda a mejorar la extensión.

Enviá un `POST` a `/api/errorlogs` con el siguiente body en JSON:

```json
{
  "error": "JSON.stringify(<objeto de error>, Object.getOwnPropertyNames(<objeto de error>))",
  "pluginVersion": "<La versión del plugin usada>"
}
```

El servicio va a guardar el siguiente documento en su base de datos:

```json
{
  "_id": "5dc89e461f8c375aa22424cc",
  "createdAt": "<fecha>",
  "error": "<objeto de error>",
  "hashedIp": "1a3a493b",
  "pluginVersion": "<versión de el plugin que usaste>",
  "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_1) AppleWebKit/537.36 (KHTML, como Gecko) Chrome/78.0.390 4.97 Safari/537.36"
}
```

### Información de versiones de las extensiones instaladas

Cuando tu navegador busca datos desde `update_url` directamente, puede que se transmitan cookies. En mi testeo, pedidos a `update_url` de extensiones descargadas desde la Chrome Web Store incluyeron los siguientes datos personales y cookies (aunque siquiera estaba logueado en mi cuenta de google):

* [1P_JAR](https://cookiepedia.co.uk/cookies/APISID/1P_JAR)
* [APISID](https://cookiepedia.co.uk/cookies/APISID/APISID)
* [HSID](https://cookiepedia.co.uk/cookies/APISID/HSID)
* [NID](https://cookiepedia.co.uk/cookies/APISID/NID)
* [SAPISID](https://cookiepedia.co.uk/cookies/APISID/SAPISID)
* [SID](https://cookiepedia.co.uk/cookies/APISID/SID)
* [SIDCC](https://cookiepedia.co.uk/cookies/APISID/SIDCC)
* [SSID](https://cookiepedia.co.uk/cookies/APISID/SSID)

(Ver también [https://policies.google.com/technologies/types](https://policies.google.com/technologies/types))

Si no querés que se transmitan estas cookies, podés usar este proxy para pedir información de versiones para extensiones instaladas. Habilitar "Aumentar privacidad" en [Notificador Chromium](https://github.com/PinkLittleKitty/chromium-notifier) hace exactamente eso.

Mandá un `POST` a `/api` con el siguiente body en JSON:

```json
{
  "prodversion": "<Versión de Chromium>",
  "extensions": [
    { "id": "<id de la extensión>",  "updateUrl": "<URL a un endpoint compatible con Omaha>"},
    { "id": "<id de la extensión>",  "updateUrl": "<URL a un endpoint compatible con Omaha>"},
    …
  ]
}
```

El servicio va a responder con un array que consistirá ded la información de versión y los metadatos de dichas extensiones:

```json
[
  {
    "codebase": "https://clients2.googleusercontent.com/crx/blobs/QgAAAC6zw0qH2DJtnXe8Z7rUJP0-NOcA97MmZN4Ln1fODAHweMXNXTmjgerLCPXhmXNXwEVIEkarzGIkPHrBXBeXqsjm4UfxBJBNpSCt104KOFaeAMZSmuWy9iapD9CEzrK8OfYl3Nvw2dw3Iw/extension_347_0_0_0.crx",
    "id": "chlffgpmiacpedhhbkiomidkjlcfhogd",
    "fp": "1.c989572d0b4c9c933f7c97224fbc49612e210b2dba994f90c87491cac53282dc",
    "hash_sha256": "c989572d0b4c9c933f7c97224fbc49612e210b2dba994f90c87491cac53282dc",
    "prodversion": "77.0.3865.90",
    "protected": "0",
    "size": "443571",
    "status": "ok",
    "timestamp": 1569175526216,
    "updateUrl": "https://clients2.google.com/service/update2/crx",
    "version": "347",
    "_id": "5d86ab09f5eeeb001cc06c34"
  },
  {
    …
  }
]
```
