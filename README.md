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
