# ESP32 - Demostración de IPC (Inter-Process Communication) con MicroPython

Este proyecto implementa una demostración de comunicación entre procesos (IPC) utilizando un **ESP32**, **MicroPython** y **`uasyncio`**. El sistema utiliza una cola (queue) para transmitir datos de un productor a un consumidor, mientras gestiona interrupciones de botones para controlar LEDs.

## 🖼️ Diagrama de Conexión (Wokwi)

![Diagrama de conexión ESP32]("C:\Users\Camilo\Desktop\SEMESTRE 5\MICROS\TEORIA\Esquema LED`s Wokwi.jpg") 

## 🔗 Enlace al Proyecto Simulado
Puedes interactuar con el esquema electrónico y simular el código en tu navegador a través de Wokwi:

👉 **[Abrir simulación en Wokwi](https://wokwi.com/projects/473195908493977601)**

## 📝 Descripción del Código

El código está diseñado para ejecutar múltiples tareas de forma asíncrona utilizando el módulo `uasyncio`. A continuación se detalla su funcionamiento por bloques:

### 1. Configuración de Pines y Hardware
Se definen los pines GPIO para los componentes:
*   **Botón 1:** GPIO 13 (Con resistencia pull-up interna).
*   **Botón 2:** GPIO 14 (Con resistencia pull-up interna).
*   **LED 1:** GPIO 25 (Salida).
*   **LED 2:** GPIO 26 (Salida).

Se inicializan los LEDs apagados y los botones con `PULL_UP`, lo que significa que al presionarlos se lee un `0` lógico (bajo).

### 2. Estructuras de Datos y Semáforos
*   **`sensor_queue`:** Una lista de Python que actúa como una cola circular (FIFO) con un límite de `QUEUE_MAX_SIZE = 10`.
*   **`button1_pressed` y `button2_pressed`:** Variables booleanas globales que actúan como semáforos/banderas para notificar que ha ocurrido una interrupción.

### 3. Interrupciones (ISR)
Se configuran las interrupciones (`irq`) en ambos botones para que se activen tanto en el flanco de subida (soltar) como en el de bajada (presionar).
*   `button1_isr` y `button2_isr` son funciones de interrupción muy rápidas que simplemente ponen su bandera correspondiente en `True`. **Importante:** No realizan operaciones complejas (como encender LEDs) dentro de la ISR para evitar bloquear el sistema.

### 4. Tareas Asíncronas (Corrutinas)
El corazón del programa se divide en 4 tareas que se ejecutan de manera concurrente gracias a `uasyncio`:

*   **`producer_task` (Productor):** Cada segundo, incrementa un contador e intenta añadirlo a la cola. Si la cola está llena (10 elementos), el dato se pierde y se imprime un mensaje de advertencia.
*   **`consumer_task` (Consumidor):** Espera a que haya datos en la cola. Si está vacía, duerme 0.1 segundos. Cuando llega un dato, lo extrae (método `pop(0)`) e imprime el valor recibido.
*   **`button1_task` y `button2_task`:** Estas tareas revisan constantemente las banderas de interrupción. Si una bandera está activa, la resetean y leen el estado actual del botón (`0` para presionado, `1` para liberado). Según el estado, encienden o apagan el LED correspondiente e imprimen el estado en la consola.

### 5. Función `main` y Ejecución
La función `main` imprime un mensaje de estado inicial y crea las 4 tareas usando `asyncio.create_task()`. Finalmente, entra en un bucle infinito `while True` con `asyncio.sleep(1)` para mantener el programa en ejecución.

## 🛠️ Requisitos para ejecutar
*   Hardware: Placa ESP32.
*   Firmware: MicroPython instalado.
*   Entorno de desarrollo: Thonny, uPyCraft o cualquier editor compatible.
*   Módulo `uasyncio` (incluido en MicroPython por defecto).

## 🚀 Cómo usar
1.  Carga el código en tu ESP32.
2.  Abre el monitor serie (a 115200 baudios).
3.  Observa los mensajes del productor y consumidor.
4.  Presiona los botones físicos (o en la simulación de Wokwi) para ver cómo los LEDs se encienden y apagan en tiempo real.
