# Vidora API

API Python con FastAPI para buscar videos y devolver URLs de reproducción mediante el reproductor oficial de YouTube.

## Qué incluye
- `GET /health`: estado del servidor.
- `GET /api/search?q=gatitos`: búsqueda de videos.
- `GET /api/videos/VIDEO_ID`: devuelve la URL oficial de inserción.
- `GET /api/trending?region=AR`: tendencias si el proveedor está disponible.
- `GET /docs`: documentación interactiva.
- `/`: página web de demostración.

## Importante sobre la reproducción
La búsqueda utiliza la API pública de instancias Invidious. Las instancias públicas pueden cambiar, limitar peticiones o dejar de funcionar. La reproducción se hace en el reproductor oficial de YouTube (`youtube-nocookie.com/embed/...`), no se descargan ni se retransmiten archivos de video. Algunos videos no permiten inserción, pueden estar restringidos por región/edad o requerir inicio de sesión. La API no elimina anuncios, no desbloquea Premium ni evita DRM.

## Ejecutar localmente (PC o Termux)
Se recomienda Python 3.10 o superior.

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

En Android/Termux también se puede probar:
```bash
pkg update
pkg install python
cd vidora_api
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8000
```
Abrí `http://127.0.0.1:8000` en el navegador del teléfono.

## Publicar gratis en Render
1. Subí esta carpeta a un repositorio de GitHub.
2. Entrá a https://render.com y conectá GitHub.
3. Elegí **New + → Web Service** y seleccioná el repositorio.
4. Runtime: Python. Build command: `pip install -r requirements.txt`.
5. Start command: `uvicorn app.main:app --host 0.0.0.0 --port $PORT`.
6. Elegí Free y creá el servicio.
7. Al terminar, Render entrega una URL HTTPS, por ejemplo `https://vidora-api.onrender.com`.
8. Probá `https://TU-URL/health` y `https://TU-URL/docs`.

Render Free puede apagar el servicio después de 15 minutos sin tráfico; el primer acceso posterior puede tardar cerca de un minuto. No es alojamiento de producción y el sistema de archivos local es efímero.

## Conectar desde Android
Usá la URL HTTPS pública de Render como `BASE_URL`.

Ejemplo Kotlin para buscar:
```kotlin
val url = "$BASE_URL/api/search?q=${Uri.encode("gatos")}"
val request = Request.Builder().url(url).build()
OkHttpClient().newCall(request).enqueue(object : Callback {
    override fun onFailure(call: Call, e: IOException) {
        runOnUiThread { /* mostrar error */ }
    }
    override fun onResponse(call: Call, response: Response) {
        val json = response.body?.string().orEmpty()
        runOnUiThread {
            // Parsear json["results"] y mostrar título, autor y thumbnail.
        }
    }
})
```

Para reproducir, tomá `embed_url` del resultado y cargalo en un WebView:
```kotlin
webView.settings.javaScriptEnabled = true
webView.settings.domStorageEnabled = true
webView.webViewClient = WebViewClient()
webView.loadUrl(videoEmbedUrl)
```
En AndroidManifest.xml asegurate de tener:
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
Para permitir reproducción en pantalla completa, implementá `WebChromeClient` y manejá `onShowCustomView`/`onHideCustomView`. No desactives la validación TLS.

## Configurar instancias de búsqueda
Si una instancia deja de funcionar, configurá en Render la variable `INVIDIOUS_INSTANCES` con una lista separada por comas de instancias confiables y activas. Consultá la lista oficial: https://docs.invidious.io/instances/


## Búsqueda avanzada (v1.1)
- Videos: `/api/search?q=rock&kind=video`
- Canales: `/api/search?q=rock&kind=channel`
- Listas: `/api/search?q=rock&kind=playlist`
- Videos cortos: `/api/search?q=baile&kind=video&duration=short`
- Videos largos: `/api/search?q=documental&kind=video&duration=long`
- Música en tendencias: `/api/trending?category=music&region=AR`
- Sugerencias: `/api/search/suggestions?q=mine`

La búsqueda de música se realiza con palabras clave y/o la categoría de tendencias de música. El filtro `duration=short` significa videos cortos por duración; no garantiza que todos sean Shorts oficiales. Invidious no expone una clasificación universal y fiable de Shorts en su endpoint general de búsqueda.
