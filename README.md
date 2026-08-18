# Sala Inteligente con ESP32 y Chatbot de Voz

Proyecto de la asignatura de Microcontroladores - Universidad Militar Nueva Granada, Programa de Ingenieria Mecatronica.

Este proyecto consiste en una maqueta de domotica donde un solo ESP32 controla toda la sala: la luz, la persiana, un piano musical y un cuadro que cambia de imagen. Todo se puede manejar por voz desde una pagina web que el mismo ESP32 sirve, asi que no hay que instalar nada en el celular ni en el computador para controlarlo.

---

## Contenido

- [Arquitectura](#arquitectura)
- [Componentes](#componentes)
- [Elementos del proyecto](#elementos-del-proyecto)
- [Conexiones](#conexiones)
- [Instalacion](#instalacion)
- [Uso](#uso)
- [Como funciona](#como-funciona)
- [Explicacion del codigo](#explicacion-del-codigo)
- [Tecnologias usadas](#tecnologias-usadas)

---

## Arquitectura

![Diagrama de bloques](docs/diagrama_bloques.png)

El ESP32 es el centro de todo el sistema. Se conecta por WiFi a una red local y desde ahi:

- Atiende las peticiones que llegan desde la pagina de voz.
- Lee constantemente los botones del piano para responder al instante cuando alguien toca una tecla.
- Controla directamente la luz, el servo de la persiana y la pantalla OLED del cuadro.

La interfaz de voz vive dentro del mismo ESP32: es una pagina web que el microcontrolador sirve en su propia direccion IP. El navegador se encarga de reconocer lo que dices y traducirlo a un comando que le manda al ESP32.

### Analisis de la estructura

El sistema sigue una arquitectura centralizada tipo estrella: todos los modulos se conectan directo al ESP32, y ninguno se comunica con otro sin pasar primero por el. Esto se puede pensar en cuatro capas:

**Percepcion y actuacion**
Es donde estan los pulsadores del piano, el modulo MOSFET de la luz, el servo de la persiana y la pantalla OLED. Son los componentes que reciben ordenes del ESP32 o le mandan informacion (en el caso de los botones).

**Control**
Es el firmware que corre dentro del ESP32. Se encarga de leer los botones del piano y de atender las peticiones que llegan por WiFi, todo en el mismo ciclo del programa, sin que una tarea tenga que esperar a que termine la otra.

**Comunicacion**
Es la conexion WiFi del ESP32 con la red local, y el servidor HTTP que corre por dentro. Gracias a esta capa, el ESP32 puede recibir ordenes desde cualquier navegador conectado a la misma red.

**Interfaz**
Es la pagina del chatbot, la parte que ve y usa la persona. No esta separada del ESP32, esta guardada dentro de el mismo, asi que no depende de internet ni de tener la pagina guardada en otro lado.

Esta forma de organizar el sistema tiene una ventaja clara: todo el control queda en un solo lugar, no hay que sincronizar varias tarjetas entre si, y si algo falla es mas facil saber donde buscar el problema porque solo hay un punto central.

---

## Componentes

| Modulo | Componentes |
|---|---|
| Luz | Modulo MOSFET Keyes + tira LED |
| Persiana | Servomotor de 360°, eje y carrete para enrollar |
| Piano | 7 pulsadores + buzzer pasivo |
| Cuadro | Pantalla OLED SSD1306 128x64 |
| Control central | ESP32 |

---

## Elementos del proyecto

**Luz inteligente**
Se armo con un modulo MOSFET (Keyes MOS Module) conectado entre el ESP32 y una tira LED. Se eligio el MOSFET en vez de conectar la tira directo al GPIO porque el pin del ESP32 no aguanta la corriente que pide la tira, y el MOSFET actua como interruptor que deja pasar esa corriente sin que el microcontrolador la reciba directamente. Responde a los comandos de encender y apagar.

**Persiana inteligente**
Usa un servomotor de 360 grados, distinto a los servos comunes que solo giran 180. Se necesitaba el rango completo para poder enrollar la persiana con un solo giro. El eje del servo mueve un carrete pequeno donde se enrolla la tela de la persiana, y del otro lado el eje se apoya suelto en un agujero del marco para que no dependa solo de la fuerza del servo.

**Piano musical**
Son 7 pulsadores, uno por cada nota de Do a Si, y un buzzer pasivo (no uno activo, porque los activos solo suenan a un tono fijo y no sirven para tocar notas distintas). Cada boton tiene asignada una frecuencia, y al presionarlo el ESP32 le manda esa frecuencia al buzzer. Funciona de forma independiente al resto del sistema, no necesita voz ni conexion a internet.

**Cuadro dinamico**
Es una pantalla OLED pequena que muestra tres imagenes distintas (un paisaje, una figura abstracta y un retrato en silueta), guardadas como mapas de bits dentro del mismo codigo. Por comando de voz se puede pasar de una imagen a la siguiente.

**Interfaz de voz**
Es la parte que conecta al usuario con todo lo demas. En vez de ser una app aparte, es una pagina web que vive guardada dentro del ESP32 y se abre escribiendo su direccion IP en cualquier navegador. Ahi se puede escribir o dictar un comando, y tambien hay botones manuales por si la voz falla.

**ESP32**
Es la tarjeta que conecta y controla todo lo anterior. Se eligio porque tiene WiFi integrado (necesario para el chatbot), suficientes pines para los 4 modulos a la vez, y soporta generar señales PWM por hardware, que es lo que se necesita para mover el servo y el buzzer sin que eso interfiera con el resto del programa.

---

## Conexiones

| Modulo | Componente | Pin ESP32 |
|---|---|---|
| Luz | MOSFET, señal | GPIO 18 |
| Persiana | Servo, señal | GPIO 19 |
| Piano | 7 pulsadores | GPIO 13, 12, 14, 27, 26, 25, 33 |
| Piano | Buzzer | GPIO 4 |
| Cuadro OLED | SDA / SCL | GPIO 21 / GPIO 22 |

---

## Instalacion

1. Instala el paquete de placas ESP32 en el Arduino IDE, en `Archivo > Preferencias > URLs adicionales`:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
2. Instala las librerias `Adafruit SSD1306` y `Adafruit GFX Library` desde el Administrador de Bibliotecas.
3. Abre `sala_inteligente_completo.ino` y cambia el nombre y la contraseña de tu red:
   ```cpp
   const char* ssid     = "NOMBRE_DE_TU_WIFI";
   const char* password = "TU_CONTRASENA";
   ```
4. Selecciona la placa ESP32 Dev Module y sube el codigo.

---

## Uso

1. Abre el Monitor Serial a 115200 baudios y espera a que el ESP32 se conecte al WiFi. Va a imprimir su IP.
2. Desde el celular o el computador, conectados a la misma red, abre un navegador y escribe esa IP.
3. Ahi carga el chatbot de voz.
4. Escribe o dicta un comando, por ejemplo:
   - enciende la luz
   - abre la persiana
   - cambia el cuadro
5. El piano funciona directo con los botones fisicos, sin necesitar voz.

---

## Como funciona

El ESP32 se conecta a la red y levanta un servidor con dos rutas: una entrega la pagina del chatbot y la otra recibe los comandos. Cuando el navegador reconoce lo que dijiste, lo compara contra unas expresiones regulares que identifican que querias hacer, y le manda al ESP32 una peticion con la accion correspondiente. El ESP32 la ejecuta y responde confirmando. El piano trabaja aparte, revisando sus botones en cada ciclo, sin depender de la voz ni de la conexion.

---

## Explicacion del codigo

El archivo `sala_inteligente_completo.ino` esta dividido en las siguientes partes:

**Variables y pines**
Al inicio se declaran los pines de cada modulo (luz, servo, teclas del piano, buzzer) y las variables de estado como `luzEncendida` o `cuadroActual`, que guardan en que posicion esta cada actuador para no perder el estado entre comandos.

**Los bitmaps del cuadro**
Los arreglos `obra1`, `obra2` y `obra3` son las tres imagenes que se muestran en la pantalla OLED. Cada una es una tabla de bytes que representa los pixeles encendidos o apagados, guardada en `PROGMEM` para no ocupar la memoria RAM del ESP32.

**La pagina web (`paginaHTML`)**
Es una cadena de texto larga que contiene todo el HTML, CSS y JavaScript del chatbot. Esta guardada dentro del mismo codigo del ESP32 para poder servirla directamente sin depender de ningun archivo externo. El JavaScript de esa pagina tiene las expresiones regulares que interpretan lo que el usuario escribe o dicta, y hace las peticiones al ESP32.

**`setup()`**
Se ejecuta una sola vez al encender el ESP32. Aqui se configuran los pines como entrada o salida, se inicializa el PWM del servo y el buzzer con `ledcAttach()`, se inicializa la pantalla OLED, se conecta el ESP32 a la red WiFi, y se definen las dos rutas del servidor (`/` y `/comando`).

**`loop()`**
Se repite todo el tiempo mientras el ESP32 esta encendido. Solo hace dos cosas en cada vuelta: revisa si llego alguna peticion al servidor (`server.handleClient()`) y revisa si alguien esta presionando una tecla del piano (`revisarPiano()`). Ninguna de las dos tareas usa `delay()`, por eso pueden funcionar al mismo tiempo sin que una bloquee a la otra.

**`manejarComando()`**
Es la funcion que se ejecuta cuando llega una peticion a `/comando`. Lee el parametro `accion` de la URL (por ejemplo `luz_on` o `persiana_abrir`), y segun cual sea, mueve el actuador correspondiente y le responde al navegador confirmando que se hizo.

**`moverServo()`**
Convierte un angulo (0 a 360 grados) en el ancho de pulso que necesita el servo, y se lo manda por PWM usando `ledcWrite()`. No usa la libreria Servo de Arduino porque no funcionaba bien con la version del ESP32 que estamos usando, asi que se calcula el pulso directamente.

**`revisarPiano()`**
Recorre los 7 pines de las teclas. Si alguna esta presionada, hace sonar la nota correspondiente en el buzzer con `ledcWriteTone()`. Si ninguna esta presionada, apaga el sonido.

**`mostrarCuadro()`**
Recibe el numero de la obra que se quiere mostrar y la dibuja en la pantalla OLED usando `drawBitmap()`.

---

## Tecnologias usadas

- Funciones `ledcAttach`, `ledcWrite` y `ledcWriteTone` para generar las señales PWM del servo y el buzzer directamente, sin librerias externas.
- Programacion no bloqueante para que el ESP32 atienda el WiFi y el piano al mismo tiempo.
- WiFi en modo estacion y servidor HTTP embebido con `WebServer.h`.
- La pagina del chatbot completa vive guardada en la memoria del ESP32 y se sirve directo al navegador.
- Modulo MOSFET para no exponer el GPIO a la corriente real de la tira LED.
- Comunicacion I2C para la pantalla OLED.

---

## Autoria

Ana Sofia Escobar - Sara Sofia Garcia

Ingenieria Mecatronica - Universidad Militar Nueva Granada
