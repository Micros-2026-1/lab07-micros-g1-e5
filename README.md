[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/rb0M7Pn8)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=23804203&assignment_repo_type=AssignmentRepo)
# Lab07: Visualización en LCD 16x2 usando módulo I²C con microcontrolador PIC

## Integrantes

* [Sebastian Suarez Garavito](https://github.com/sebastianavas1701-stack)
* [Sebastian Buitrago Oliveros](https://github.com/SebastianBuitrago-16)
* [Samuel Esteban Jaime Gutierrez](https://github.com/Samueljgest)
## Documentación
Para el desarrollo del laboratorio se consultaron las hojas de datos del microcontrolador PIC18F utilizado, donde se especifican las características del módulo MSSP (Master Synchronous Serial Port) necesario para la implementación del protocolo I²C, incluyendo los registros de configuración SSPCON1, SSPCON2, SSPSTAT y SSPBUF, así como la asignación de los pines SCL y SDA correspondientes.

Adicionalmente, se revisó la documentación del módulo LCD basado en el controlador HD44780 y su adaptador I²C PCF8574, con el fin de comprender el direccionamiento del dispositivo, el protocolo de inicialización en modo 4 bits y la estructura de los comandos para control del cursor, escritura de caracteres y carga de caracteres personalizados en la CGRAM.

Se consultó también la documentación del PICkit 4 para la programación y depuración del microcontrolador, junto con el entorno MPLAB X IDE, utilizado para la compilación y carga del firmware. Por último, se apoyó el proceso con las guías de laboratorio y el repositorio del curso en GitHub Classroom, donde se encontraban los archivos base del proyecto.
## Diagramas

## Evidencias de implementación

## Preguntas

1. ¿Por qué I²C se clasifica como half-duplex mientras que SPI es full-duplex? ¿Qué implicación práctica tiene esa diferencia para el control de una LCD?.

RESPUESTA: 
    I²C utiliza una única línea bidireccional de datos (SDA) compartida entre maestro y esclavos, lo que significa que la transmisión y recepción no pueden ocurrir simultáneamente, clasificándolo como half-duplex. SPI, en cambio, dispone de líneas separadas para MOSI y MISO, permitiendo comunicación simultánea en ambas direcciones (full-duplex).
    
    En el control de una LCD esta diferencia no representa un inconveniente crítico, ya que la pantalla es esencialmente un dispositivo de escritura: el microcontrolador envía comandos y datos, pero no necesita recibir información de retorno en tiempo real. Por tanto, el modo half-duplex de I²C es suficiente para esta aplicación.

2. En I2C_init() se asigna SSPCON1 = 0x28. Desglose ese valor bit a bit e identifique qué modo de operación del MSSP se está seleccionando y por qué se elige ese valor.

    RESPUESTA: 
3. Las funciones I2C_start(), I2C_stop() e I2C_write() comparten el mismo patrón: activar un bit de control y luego esperar con while(!PIR1bits.SSPIF). ¿Qué representa la bandera SSPIF y por qué se limpia después de cada operación?.

RESPUESTA: 
    
    La bandera SSPIF (SSP Interrupt Flag) del registro PIR1 se activa automáticamente por hardware cada vez que el módulo MSSP completa una operación, ya sea una condición de inicio, parada o la transmisión de un byte. El ciclo while(!PIR1bits.SSPIF) funciona como una espera activa que bloquea la ejecución hasta que dicha operación finaliza, garantizando sincronización entre el software y el hardware.

    Limpiarla manualmente con PIR1bits.SSPIF = 0 es indispensable porque el hardware no la resetea solo: si no se limpia, la siguiente operación encontraría la bandera ya activa y no esperaría correctamente, corrompiendo la secuencia de comunicación.

4. El fuse PBADEN = OFF está presente en la configuración. ¿Qué efecto tendría dejarlo en ON sobre los pines del puerto B, y por qué podría causar problemas si se usan esos pines como salidas digitales?.

RESPUESTA: 

    El fuse PBADEN controla la función inicial de los pines RB0 a RB4 al arrancar el microcontrolador. Con PBADEN = ON, estos pines se configuran por defecto como entradas analógicas, activando el módulo ADC sobre ellos. Si en el código se intenta usarlos como salidas digitales sin desactivar explícitamente la función analógica mediante el registro ANSELB, el pin no responderá como salida digital, permaneciendo en alta impedancia y sin generar los niveles lógicos esperados. Esto puede causar un comportamiento impredecible en cualquier periférico conectado a ese puerto, razón por la cual se establece PBADEN = OFF para que los pines inicien directamente en modo digital.
5. Compare el control de la LCD en modo paralelo (lab04) con el modo I²C de este laboratorio. Mencione ventajas y desventajas de cada enfoque en términos de: cantidad de pines usados, velocidad de actualización y complejidad del código.

RESPUESTA:


6. El bus I²C permite conectar múltiples esclavos con solo dos hilos. Si se quisiera agregar un segundo módulo PCF8574 al mismo bus (por ejemplo, para controlar un segundo LCD), ¿qué cambio mínimo sería necesario en el hardware y en el código?

RESPUESTA:

    El bus I²C identifica cada dispositivo mediante una dirección de 7 bits. El módulo PCF8574 permite configurar su dirección a través de los pines físicos A0, A1 y A2. Por tanto, el cambio mínimo necesario sería:

* En hardware: modificar la combinación de los pines A0–A2 del segundo PCF8574 para asignarle una dirección distinta a la del primero (por ejemplo, si el primero tiene dirección 0x4E, el segundo podría configurarse en 0x4C).
*En código: definir una segunda constante de dirección y crear funciones o una versión parametrizada de lcd_cmd y lcd_write_char que reciban la dirección destino como argumento, de modo que el maestro pueda dirigirse a cada LCD de forma independiente usando el mismo bus SCL/SDA.

## Conclusiones

## Referencias