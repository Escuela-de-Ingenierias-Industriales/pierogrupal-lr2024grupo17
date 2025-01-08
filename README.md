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

![image](https://github.com/user-attachments/assets/932be7aa-ba15-466c-b2ab-3d91860f9050)

La otra opción sería programando el PIERO con una función de programación de bajo nivel en Simulink. Esta opción utiliza la herramienta S-Function Builder, almacenando un programa en C que permite hacer la lectura de los encoders. Esto se ha probado con la función proporcionada en los vídeos de clase, pero al final se ha optado por la solución de la lectura de los encoders.
