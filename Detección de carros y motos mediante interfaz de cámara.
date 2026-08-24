# ESP32-WROOM-32: Control de LEDs con Pulsadores y Detección de Vehículos con YOLO

Este repositorio integra dos proyectos complementarios en una sola plataforma:

1. **Control de LEDs por pulsadores** utilizando las capacidades de GPIO de la ESP32-WROOM-32, demostrando el manejo de entradas/salidas digitales con interrupciones y FreeRTOS.

2. **Detección de vehículos (autos y motos)** mediante un modelo YOLO (You Only Look Once) entrenado previamente, ejecutado en un entorno virtual para demostrar aplicaciones de visión por computadora en tiempo real, con integración a la ESP32 mediante comunicación serial.

Ambos proyectos comparten el mismo ecosistema de desarrollo, mostrando la versatilidad de la ESP32 para aplicaciones embebidas y de inteligencia artificial en el borde.

---

## Práctica 1: Control de LEDs con Pulsadores

### 1.1 Descripción

Esta práctica consiste en implementar un sistema de control de LEDs mediante pulsadores físicos utilizando la ESP32-WROOM-32. Se hace uso de las capacidades de entrada y salida digital de la placa, implementando interrupciones por flanco y comunicación entre tareas mediante FreeRTOS.

### 1.2 Componentes y Conexiones

| Componente | Puerto GPIO | Función |
|------------|-------------|---------|
| Pulsador 1 | GPIO 13 | Encender/Apagar LED 1 |
| Pulsador 2 | GPIO 14 | Encender/Apagar LED 2 |
| LED 1 (Rojo) | GPIO 25 | Indicador de estado |
| LED 2 (Verde) | GPIO 26 | Indicador de estado |

### 1.3 Diagrama de Conexiones

![Esquema de conexiones ESP32 - LEDs y Pulsadores](Esquema%20LEDs%20Wokwi.jpg)

*Figura 1: Diagrama de conexiones para la Práctica 1 - Control de LEDs con pulsadores*

### 1.4 Funcionamiento

- **Pulsador 1 (GPIO13)** → Enciende/Apaga **LED 1 (GPIO25)**
- **Pulsador 2 (GPIO14)** → Enciende/Apaga **LED 2 (GPIO26)**

El sistema utiliza interrupciones por flanco (ANYEDGE) para detectar cuando se presiona o suelta cada botón. Los pulsadores están configurados con resistencias de pull-up internas, por lo que:
- Estado LOW (0) → Botón presionado
- Estado HIGH (1) → Botón liberado

### 1.5 Tecnologías Utilizadas

- **Microcontrolador:** ESP32-WROOM-32
- **Lenguaje:** C (ESP-IDF)
- **Sistema Operativo:** FreeRTOS
- **Mecanismos de IPC:** Semáforos y Colas
- **Simulación:** Wokwi

### 1.6 Estructura del Código

El código implementa las siguientes tareas FreeRTOS:

| Tarea | Función | Prioridad |
|-------|---------|-----------|
| Producer Task | Genera datos de sensor y los envía a la cola | 3 |
| Consumer Task | Recibe datos de la cola y los procesa | 2 |
| Button 1 Task | Espera semáforo del botón 1 y controla LED 1 | 4 |
| Button 2 Task | Espera semáforo del botón 2 y controla LED 2 | 4 |

### 1.7 Código Principal

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "freertos/semphr.h"
#include "driver/gpio.h"

/* Definiciones de GPIO */
#define BUTTON1_GPIO    GPIO_NUM_13
#define BUTTON2_GPIO    GPIO_NUM_14
#define LED1_GPIO       GPIO_NUM_25
#define LED2_GPIO       GPIO_NUM_26

/* Objetos IPC */
QueueHandle_t sensorQueue;
SemaphoreHandle_t button1Semaphore;
SemaphoreHandle_t button2Semaphore;

/* ISR para Button 1 */
static void IRAM_ATTR button1_isr_handler(void *arg)
{
    BaseType_t higherPriorityTaskWoken = pdFALSE;
    xSemaphoreGiveFromISR(button1Semaphore, &higherPriorityTaskWoken);
    if (higherPriorityTaskWoken) portYIELD_FROM_ISR();
}

/* ISR para Button 2 */
static void IRAM_ATTR button2_isr_handler(void *arg)
{
    BaseType_t higherPriorityTaskWoken = pdFALSE;
    xSemaphoreGiveFromISR(button2Semaphore, &higherPriorityTaskWoken);
    if (higherPriorityTaskWoken) portYIELD_FROM_ISR();
}

/* Tarea Productora */
void producer_task(void *pvParameters)
{
    int count = 0;
    while (1) {
        count++;
        printf("[PRODUCER] Sending sensor data: %d\n", count);
        if (xQueueSend(sensorQueue, &count, portMAX_DELAY) == pdPASS) {
            printf("[PRODUCER] Data sent successfully\n");
        }
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

/* Tarea Consumidora */
void consumer_task(void *pvParameters)
{
    int received_val;
    while (1) {
        if (xQueueReceive(sensorQueue, &received_val, portMAX_DELAY)) {
            printf("[CONSUMER] Received Sensor Data: %d\n", received_val);
        }
    }
}

/* Tarea para Button 1 */
void button1_task(void *pvParameters)
{
    while (1) {
        if (xSemaphoreTake(button1Semaphore, portMAX_DELAY)) {
            int button_state = gpio_get_level(BUTTON1_GPIO);
            if (button_state == 0) {
                gpio_set_level(LED1_GPIO, 1);
                printf("[BUTTON 1] PRESSED -> LED1 ON\n");
            } else {
                gpio_set_level(LED1_GPIO, 0);
                printf("[BUTTON 1] RELEASED -> LED1 OFF\n");
            }
        }
    }
}

/* Tarea para Button 2 */
void button2_task(void *pvParameters)
{
    while (1) {
        if (xSemaphoreTake(button2Semaphore, portMAX_DELAY)) {
            int button_state = gpio_get_level(BUTTON2_GPIO);
            if (button_state == 0) {
                gpio_set_level(LED2_GPIO, 1);
                printf("[BUTTON 2] PRESSED -> LED2 ON\n");
            } else {
                gpio_set_level(LED2_GPIO, 0);
                printf("[BUTTON 2] RELEASED -> LED2 OFF\n");
            }
        }
    }
}

/* Inicialización de GPIO */
void gpio_init(void)
{
    /* Configurar LEDs como salida */
    gpio_config_t led_config = {
        .pin_bit_mask = (1ULL << LED1_GPIO) | (1ULL << LED2_GPIO),
        .mode = GPIO_MODE_OUTPUT,
        .pull_up_en = GPIO_PULLUP_DISABLE,
        .pull_down_en = GPIO_PULLDOWN_DISABLE,
        .intr_type = GPIO_INTR_DISABLE
    };
    gpio_config(&led_config);
    gpio_set_level(LED1_GPIO, 0);
    gpio_set_level(LED2_GPIO, 0);

    /* Configurar botones con pull-up e interrupción en ambos flancos */
    gpio_config_t button_config = {
        .pin_bit_mask = (1ULL << BUTTON1_GPIO) | (1ULL << BUTTON2_GPIO),
        .mode = GPIO_MODE_INPUT,
        .pull_up_en = GPIO_PULLUP_ENABLE,
        .pull_down_en = GPIO_PULLDOWN_DISABLE,
        .intr_type = GPIO_INTR_ANYEDGE
    };
    gpio_config(&button_config);
}

/* Aplicación Principal */
void app_main(void)
{
    printf("\n========================================\n");
    printf(" ESP32 FreeRTOS IPC Demonstration\n");
    printf("========================================\n");

    gpio_init();
    printf("[INIT] GPIO initialized\n");

    /* Crear cola */
    sensorQueue = xQueueCreate(10, sizeof(int));
    if (sensorQueue == NULL) {
        printf("[ERROR] Queue creation failed!\n");
        return;
    }
    printf("[INIT] Sensor queue created\n");

    /* Crear semáforos */
    button1Semaphore = xSemaphoreCreateBinary();
    button2Semaphore = xSemaphoreCreateBinary();
    if (button1Semaphore == NULL || button2Semaphore == NULL) {
        printf("[ERROR] Semaphore creation failed!\n");
        return;
    }
    printf("[INIT] Button semaphores created\n");

    /* Instalar servicio de ISR */
    gpio_install_isr_service(0);
    printf("[INIT] GPIO ISR service installed\n");

    /* Adjuntar ISRs */
    gpio_isr_handler_add(BUTTON1_GPIO, button1_isr_handler, NULL);
    gpio_isr_handler_add(BUTTON2_GPIO, button2_isr_handler, NULL);
    printf("[INIT] Button interrupts attached\n");

    /* Crear tareas */
    xTaskCreate(producer_task, "Producer Task", 2048, NULL, 3, NULL);
    xTaskCreate(consumer_task, "Consumer Task", 2048, NULL, 2, NULL);
    xTaskCreate(button1_task, "Button1 Task", 2048, NULL, 4, NULL);
    xTaskCreate(button2_task, "Button2 Task", 2048, NULL, 4, NULL);
    printf("[INIT] All FreeRTOS tasks created\n");

    printf("----------------------------------------\n");
    printf("Button 1 : GPIO13\n");
    printf("Button 2 : GPIO14\n");
    printf("LED 1    : GPIO25\n");
    printf("LED 2    : GPIO26\n");
    printf("----------------------------------------\n");
    printf("[READY] System ready!\n");

    while (1) {
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
