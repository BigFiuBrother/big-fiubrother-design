# Big Fiubrother

# Resumen

Big Fiubrother es un sistema de vigilancia a través de multiples cámaras de
video. El video es procesado para encontrar a personas registradas dentro del
sistema e informar si se encuentran intrusos ajenos al establecimiento. Toda
esta información es adjuntada al video y se podrá observar a través de una
aplicación web.

Un video pasa por multiples etapas de procesamiento hasta considerarse
procesado. Estas etapas consisten en:
1. Particionamiento: Un stream de video capturado por una cámara es dividido 
en un video de _S_ segundos de duración.
2. Muestreo: El video es dividido en imágenes, de las cuales un grupo son
seleccionadas a una tasa fija de muestreo _SR_.
3. Detección: Las imágenes son procesadas por un algoritmo de reconocimiento
facial para detectar rostros humanos en la imagen.
4. Clasificación: Los rostros detectados son procesados por un algoritmo de 
clasificación para detectar a las personas que el sistema conoce.

Dado que el procesamiento de video es el núcleo del proyecto, se buscará optimizar el
diseño para que se pueda escalar horizontalmente. Cada una de las etapas deberá poder
trabajar en la misma o distintas computadoras y podrá haber más de un _worker_ por etapa.

Para garantizar el procesamiento en tiempo real de la información el sistema debera poder
controlar dinámicamente el poder de cómputo disponible, incrementando la cantidad de _workers_ 
destinada al procesamiento de cada etapa.


En cuanto a la aplicación web, esta se encargará de:
 - Mostrar el stream de video recibido por la/s cámaras. El video contará con
la información de procesamiento, indicando a las personas registradas y a los
intrusos.
 - Agregar/Modificar/Eliminar personas registradas.
 - Mostrar un historial de los últimos movimientos de una persona seleccionada.


# Plan de Desarrollo

## N Cámaras, M computadoras, un servidor web

N Cámaras
- Particionamiento de video stream
- Publicación de Video en Servidor
- Publicación de Evento _Video para procesamiento_

M Computadores
- Procesamiento de Video (2, 3, 4) en paralelo
- Recibe evento de nuevo video
- Publicacion de rostros en servidor

Servidor web
- Hostea aplicacion web
- Recibe eventos de nuevos y nuevos rostros

## Dynamic Balancer

Si se puede ejecutar todos los modulos en paralelo independientemente, entonces:

```
Si |V| + |P| <= |D|, entonces el sistema es fluido.
Donde
|V| es la duración del fragmento de video.
|P| es el tiempo de procesamiento del fragmento de video, desde el muestreo hasta
que todos los frames y caras sean procesados.
|D| es el delay inicial del sistema.
```

El sistema de balanceo tiene dos objetivos:
- Agregar más workers de la etapa necesaria cuando la carga es mayor que lo que se
puede procesar.
- Bajar recursos cuando la carga es menor a la esperada.

# Variable Fijas

- Duración de fragmento de video (Menor a 10 segundos)
- Delay (Menor a 10 segundos y mayor a duración de video)


# Variables Dinámicas

- Cantidad de workers de muestreo _WS_
- Cantidad de workers de detección _WR_
- Cantidad de workers de clasificación _WC_

```
|LS| + |TS| + |LD| + |TD| + |TR| + |LR| <= |P|
```

Throughput de una cola de mensajes para una ventana de tiempo |T|
```
|ME| durante |T| =~ |MS| durante |T|
(|ME| * W / |MS|) - |W| = |W a agregar|
```


Poder de procesamiento de una etapa E para una ventana de tiempo |T|
```
|WEi| * |T| =~ Sum of |TEi| during |T|
|T| >> |V|
```


<div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers tags lightbox&quot;,&quot;edit&quot;:&quot;_blank&quot;,&quot;xml&quot;:&quot;&lt;mxfile host=\&quot;app.diagrams.net\&quot; modified=\&quot;2022-08-29T22:04:29.266Z\&quot; agent=\&quot;5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.5112.102 Safari/537.36\&quot; etag=\&quot;0Kl_cd3YSC6WMl9VE1GP\&quot; version=\&quot;20.2.7\&quot; type=\&quot;google\&quot;&gt;&lt;diagram id=\&quot;6y4UHZnNiBQ1XcxcdRBw\&quot; name=\&quot;Page-1\&quot;&gt;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&lt;/diagram&gt;&lt;/mxfile&gt;&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/js/viewer-static.min.js"></script>
