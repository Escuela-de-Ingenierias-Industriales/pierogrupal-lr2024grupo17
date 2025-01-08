# Estudio de la plataforma robótica móvil PIERO

En primer lugar, se investigaron los componentes que forman la plataforma robótica móvil PIERO, los cuales se mencionan a continuación:

1. Ruedas, motores y encoders: las ruedas usadas son de goma, de 65mm de diámetro, con casquillos de 4mm (eje). Los motores tienen una velocidad nominal de 170rpm, trabajarán a 12V, contando con encoders incorporados. Estos encoders convertirán el movimiento de las ruedas en una señal eléctrica, que luego se podrá leer a través de una conexión con Arduino y Simulink.
2. Sensores de distancia: se han usado sensores de distancia de ultrasonidos, para poder detectar los obstáculos que estén cercanos al PIERO.
3. Led: se ha utilizado un LED RGB con tres pines, como método de señalización.
4. Baterías: se han usado una serie de baterías de litio, de 3.7V, para alimentar el sistema del PIERO.
5. Voltímetro: se ha utilizado para controlar la descarga de la batería.
6. Sensor de voltaje: va a permitir captar el voltaje en el que se está trabajando (12V), ya que la placa de Arduino solo es capaz de captar hasta 5V, por defecto.
7. Driver de potencia L298N: cuenta con un puente H, que permite el control de la velocidad y el sentido de giro de las ruedas que están con los motores. También ofrece unos pines dedicados a una señal de "Enable", que permite habilitar o deshabilitar los motores. Tiene protecciones contra corriente, temperatura y corrientes inducidas. Es capaz de trabajar desde los 3V, hasta los 35V.
8. Arduino MEGA 2560: microcontrolador que permitirá programar el robot móvil, está basado en la ATmega2560. Cuenta con 54 pines de I/O (entrada/salida), de los cuales 15 pueden funcionar como PWM. Además, tiene 16 pines de entrada analógica y 4 pines de comunicación serie (UART). En cuanto a memoria fija y variable, posee una memoria flash de 256kB y una memoria SRAM de 8kB. Por otra parte, posee un puerto USB que se puede conectar directamente al ordenador, y que trabaja a 5V.

# Entradas y salidas, I/O
Una vez que el robot ya está montado y los componentes conectados, se crean varios modelos que permiten simular las entradas y salidas del sistema.

## Sensores de distancia: Sonars
El esquema realizado en Simulink es el siguiente:

![image](https://github.com/user-attachments/assets/5073361d-af7f-4744-b8dc-f802df7d47ba)

Como se puede ver en la imagen, se han usado dos bloques de la librería de Arduino, junto con un bloque de ganancia. Estos son bloques de entrada analógica, que reciben los datos de voltaje del sónar de la izquierda (pin A5) y de la derecha (pin A4). Luego, se ha conectado un bloque de ganancia para poder convertir ese voltaje a centímetros.

## Motores
A continuación, se explica el subsistema de los motores.

![image](https://github.com/user-attachments/assets/cac09d6c-b275-44e7-940f-5965c3863ae7)

Se han usado bloques de salida de PWM, junto con bloques de salida digital para las señales de "enable" de cada motor (derecho e izquierdo). La entrada a esos bloques tiene que estar limitada entre 0 y 255 para cuando las ruedas van hacia delante, y de -255 a 0 cuando van hacia atrás. Es por esto por lo que se han colocado los bloques de "Saturation". Como el PWM solo maneja valores positivos, se ha colocado un bloque que calcula el valor absoluto de la señal en los casos en los que se quiera ir hacia atrás.

## LED
En este apartado se tienen dos modelos: el primer modelo sirve para elegir el color del led según la entrada que se reciba, y el segundo modelo sirve como un sistema de señalización para detectar obstáculos cercanos.

El modelo que enciende el led de un color u otro, según los bits de la señal de entrada, es el siguiente.

![image](https://github.com/user-attachments/assets/312d3381-0e1a-431d-98de-592c4a0f8fef)


Por otra parte, el sistema de señalización es el siguiente. 

![image](https://github.com/user-attachments/assets/e315b281-ad77-48cb-acff-5f08661cd9f6)

## Navegación reactiva básica
Con los bloques explicados anteriormente, se crea un primer modelo de navegación reactiva para el robot PIERO.

![image](https://github.com/user-attachments/assets/3ca669f5-6ca8-4fc8-a5a0-81ace7bfe3db)

Se utiliza el bloque de los sensores de ultrasonidos de entrada, que entra al bloque de señalización de los leds, para luego pasar al bloque que enciende el led RGB.

# Lectura de los codificadores, y las funciones de programación de bajo nivel en Simulink (sfunction)

Hay varias opciones disponibles para la lectura de los codificadores. Una de ellas es leer directamente con un bloque de "encoder" de la librería de Arduino en Simulink, describiendo el número de pin que corresponde a cada encoder. Esta opción es la que se ha usado para este proyecto. El modelo de Simulink creado para esto se puede ver a continuación. Del bloque del encoder salen los "ticks" leídos por el codificador.

![image](https://github.com/user-attachments/assets/bbd55319-1905-446c-923a-3466c3eec95c)

Se ha creado otro modelo donde se añade un bloque de derivada discreta. Esto permite que se pueda convertir los flancos del encoder en velocidad en metros por segundo.

# Identificación y simulación de los sistemas motor, comunicaciones serie y generador de señales en Simulink

# Generador de señales en Simulink
Para la simulación de los sistemas motor, se han usado dos tipos de señales creadas con el bloque "Signal Builder". Las dos señales se pueden observar en las imágenes a continuación.

La primera de las señales ha sido una señal rampa.

![image](https://github.com/user-attachments/assets/7c3d587e-ddff-4816-8d2b-f0f4d1a48d45)

La segunda ha sido un tren de pulsos.

![image](https://github.com/user-attachments/assets/972f6867-54b0-48ce-8cc0-8fba94cec83c)


# Identificación y Simulación de los sistemas motor
Para simular los sistemas se ha creado este modelo en Simulink.

![image](https://github.com/user-attachments/assets/c876555f-29b7-4974-b914-740155148c03)

Este modelo contiene el bloque de "SignalBuilder" comentado anteriormente, junto con un subsistema llamado "PieroHW", que se corresponde con el modelo de Hardware del PIERO, estudiado en los puntos anteriores. También se han añadido unos bloques que guardan los datos de velocidad y PWM procedentes del PIERO. Estos datos son introducidos en la aplicación "System Identification", tanto los de la rueda derecha como la izquierda. Una vez hecho, se ve qué funciones de transferencia funcionan mejor, y se seleccionan y reúnen los coeficientes para las funciones de transferencia discretas de cada rueda.

![image](https://github.com/user-attachments/assets/05b44d5c-1349-4995-b507-ecb53b27dcc0)

Reunidos estos datos, creamos un modelo que permita elegir si se está en modelo de Piero Hardware, o en modelo de Piero con las funciones de transferencia. Este modelo es el indicado a continuación:

![image](https://github.com/user-attachments/assets/64e0e8e0-f09d-4883-be5c-3b7e4904565d)

![image](https://github.com/user-attachments/assets/932be7aa-ba15-466c-b2ab-3d91860f9050)

La otra opción sería programando el PIERO con una función de programación de bajo nivel en Simulink. Esta opción utiliza la herramienta S-Function Builder, almacenando un programa en C que permite hacer la lectura de los encoders. Esto se ha probado con la función proporcionada en los vídeos de clase, pero al final se ha optado por la solución de la lectura de los encoders.


# Bucle Abierto

Ahora, se integra todo en un sistema de bucle abierto, que contiene las partes de Hardware y Software del PIERO.

El modelo del bucle abierto se puede ver a continuación, con el controlador en bucle abierto y el modelo de piero hardware ya mencionado anteriormente:

![image](https://github.com/user-attachments/assets/54fdc3f7-0137-427e-9585-bbbc02107f94)

A continuación se ve el controlador para el bucle abierto. Para dicho controlador se han utlizado dos tablas de consulta (Look-Up Tables), una para cada rueda del robot móvil.

![image](https://github.com/user-attachments/assets/12066d33-f509-4d55-94b7-1718c332016c)


# Bucle Cerrado

En este modelo, se cambian las Look-Up Tables por las funciones de transferencia correspondientes a cada rueda. Además, se introduce una realimentación para reducir el error que se obtenía en el modelo de bucle abierto.

![image](https://github.com/user-attachments/assets/d2ead320-e77d-487d-ad01-40263237ce6a)


# Cinemática

Teniendo en cuenta los modelos construidos anteriormente, se construyen los modelos cinemático directo (MCD) e inverso (MCI) del PIERO, junto con la odometría.
Con el modelo cinemático inverso se obtienen las velocidades de ambas ruedas a partir de las velocidades lineal y angular, mientras que el modelo cinemático directo hace la operación inversa. Luego se tiene el modelo de Odometría, que junto con los bloques anteriores permite estimar la posición y orientación del robot móvil.

Modelo de cinemática completo: la entrada son las consignas de velocidad lineal y angular. Luego, se aplica el modelo cinemático inverso para obtener velocidades articulares y que el PIERO pueda leerlas. Después, se aplica el modelo cinemático directo para volver a obtener las velocidades cartesianas y obtener una representación en el plano XY del movimiento gracias a la odometría.

![image](https://github.com/user-attachments/assets/8005896c-c9c1-40ea-927a-56a910c8787d)

Modelo cinemático inverso (MCI): convierte velocidades cartesianas en velocidades articulares.

![image](https://github.com/user-attachments/assets/801b69dc-9036-47a7-98d3-8863d6a74b23)

Modelo cinemático directo (MCD): convierte velocidades articulares en velocidades cartesianas.

![image](https://github.com/user-attachments/assets/5e3a5b8b-bc85-458d-89ae-61954b52cb6b)

Sistema de Odometría: obtiene una representación en un plano XY del movimiento del robot móvil, por lo que permite observar la trayectoria del robot. Este bloque se basa en las siguientes ecuaciones:
Vx = v*cos(v)   ;   Vy = v*sin(v)   ;   w = w

![image](https://github.com/user-attachments/assets/6aaf62ce-aad0-474f-b888-2a31003121d1)

# Control de Trayectorias

