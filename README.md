# Evaluación IoT de iluminación en un invernadero

Este proyecto corresponde al caso nuevo de la evaluación individual. Los valores de luz son **simulados**: no se necesita ESP32, DHT22 ni un montaje en Wokwi. Se envían por MQTT al tópico `iot/invernadero/iluminacion`. Mosquitto es el broker y Node-RED se suscribe, valida el dato, marca el estado, lo añade a un CSV y sirve un dashboard local.

## Archivos que deben estar juntos

- `compose.yaml`
- `mosquitto.conf`
- `flows.json`

`README.md` también puede estar en esa carpeta, pero Docker no lo necesita.

## Preparación en Windows

1. Crea una carpeta, por ejemplo `InvernaderoIoT`, y descarga ahí los tres archivos anteriores. Verifica que Windows no haya añadido `.txt` al final de los nombres.
2. Abre Docker Desktop y espera a que el motor esté en ejecución.
3. Abre PowerShell **en esa misma carpeta**. Puedes escribir `powershell` en la barra de dirección del Explorador y pulsar Enter.
4. Ejecuta:

   ```powershell
   docker compose up -d
   docker compose ps
   ```

   Deben aparecer `broker` y `nodered` ejecutándose. La primera descarga de las imágenes puede tardar.

5. Abre <http://localhost:1880>. En el menú de tres líneas de Node-RED selecciona **Import**, elige `flows.json`, importa el flujo y pulsa **Deploy**. Haz esto **solo una vez**. La conexión del nodo MQTT debe indicar `connected`.
6. Abre <http://localhost:1880/invernadero>. La pantalla mostrará «ESPERANDO LECTURAS» hasta la primera publicación.

## Publicar y comprobar todo el recorrido

Abre dos ventanas de PowerShell en la carpeta del proyecto. En la **primera**, inicia un suscriptor de prueba:

```powershell
docker compose exec broker mosquitto_sub -h localhost -t iot/invernadero/iluminacion -v
```

Déjalo esperando. En la **segunda**, publica las cinco lecturas simuladas:

```powershell
180, 350, 520, 750, 900 | ForEach-Object {
    docker compose run --rm nodo -h broker -t iot/invernadero/iluminacion -m "$_" -q 1 -d
    Start-Sleep -Seconds 2
}
```

El suscriptor debe mostrar el tópico y cada número. El dashboard actualizará la última lectura y la tabla. En el editor de Node-RED, abre la pestaña **Debug** para ver los objetos procesados.

Consulta el registro que escribió Node-RED:

```powershell
docker compose exec nodered cat /data/lecturas.csv
```

Cada fila contiene fecha ISO, lux y estado. Si no existe todavía, publica un valor y vuelve a comprobar.

## Reto de 1250 lux

```powershell
docker compose run --rm nodo -h broker -t iot/invernadero/iluminacion -m 1250 -q 1 -d
```

El suscriptor debe recibir `1250`. El dashboard debe mostrar **1250 lux** y **ALERTA**; también aparecerá `1250,ALERTA` en una fila nueva del CSV. La regla está en el nodo **Validar y procesar** de Node-RED, con un umbral **ilustrativo de 1000 lux**. El enunciado pide explicar dónde se colocaría una regla, sin fijar un umbral exacto.

## Evidencias para tomar en tu laptop

1. `docker compose ps`: broker y Node-RED en ejecución.
2. Dos terminales: publicación de valores y recepción de esos mismos valores mediante `mosquitto_sub`.
3. Flujo de Node-RED con el nodo MQTT conectado y pestaña Debug mostrando una lectura procesada.
4. Dashboard abierto con lecturas normales y otra captura tras publicar 1250 lux.
5. Salida del CSV con las lecturas guardadas.

Las capturas deben ser de tu ejecución. No presentes una maqueta ni los valores como mediciones físicas reales.

## Explicación oral breve

«Simulé el nodo de iluminación publicando valores en `iot/invernadero/iluminacion`. El broker Mosquitto recibe los mensajes y los distribuye a los clientes suscritos. Node-RED se suscribe a ese tópico, valida la lectura, compara los lux con un umbral de ejemplo, guarda cada fila en CSV y entrega los datos al dashboard. Por eso puedo seguir el mismo 1250 desde el publicador, pasando por el suscriptor y el procesamiento, hasta la pantalla. Si la pantalla no cambia, revisaría primero la suscripción, el tópico y el Debug de Node-RED».

## Si algo no aparece

- Confirma que Docker Desktop esté activo y repite `docker compose ps`.
- Revisa `docker compose logs --tail=50 broker nodered`.
- En Node-RED confirma que importaste el flujo, pulsaste **Deploy** y MQTT aparece como `connected`.
- El tópico debe coincidir exactamente, incluida la barra y la ortografía: `iot/invernadero/iluminacion`.
- Abre el dashboard en **`/invernadero`**, no en la portada del editor.
- Si el puerto 1880 ya está ocupado, habrá que modificar el puerto local del servicio `nodered` en `compose.yaml`.

Al terminar puedes pulsar `Ctrl+C` en el suscriptor y detener los servicios con `docker compose down`. Ese comando conserva el volumen con el CSV y el flujo de Node-RED.

## Entrega de la evaluación

- `compose.yaml`, `mosquitto.conf` y `flows.json`: configuración y flujo del prototipo.
- `datos/lecturas.csv`: lecturas registradas durante la prueba.
- `docs/Informe_Laboratorio_IoT_Invernadero_Diego_Ventura.docx`: informe con capturas, explicación del recorrido del dato y cuestionario adaptado.

Se probaron las lecturas de 180, 350, 520, 750 y 900 lux. La prueba adicional de 1250 lux llegó al dashboard y fue clasificada como ALERTA mediante un umbral de demostración de 1000 lux.