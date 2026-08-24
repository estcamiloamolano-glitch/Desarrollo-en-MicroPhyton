# ESP32-WROOM-32 + YOLO: Control y Visión por Computadora

Este repositorio integra dos proyectos complementarios en una sola plataforma:

1. **Control de LEDs por pulsadores** utilizando las capacidades de GPIO de la ESP32-WROOM-32.
2. **Detección de vehículos (autos y motos)** mediante un modelo YOLO pre-entrenado.

---

##    Práctica 1: Control de LEDs con Pulsadores

### Diagrama de Conexiones

<div align="center">
  <img src="Esquema%20leds.jpg" alt="Diagrama de conexiones ESP32" width="700">
  <p><em>Figura 1: Diagrama de conexiones para la Práctica 1</em></p>
</div>

### Componentes y Puertos

| Componente | Puerto GPIO | Función |
|------------|-------------|---------|
| Pulsador 1 | GPIO 13 | Encender/Apagar LED 1 |
| Pulsador 2 | GPIO 14 | Encender/Apagar LED 2 |
| LED 1 (Rojo) | GPIO 25 | Indicador de estado |
| LED 2 (Verde) | GPIO 26 | Indicador de estado |

### Funcionamiento

- **Pulsador 1 (GPIO13)** → Enciende/Apaga **LED 1 (GPIO25)**
- **Pulsador 2 (GPIO14)** → Enciende/Apaga **LED 2 (GPIO26)**

El sistema utiliza interrupciones por flanco para detectar cuando se presiona o suelta cada botón.

---

##    Práctica 2: Detección de Vehículos con YOLO

### Descripción

Sistema de visión por computadora que utiliza el modelo **YOLO (You Only Look Once)** para detectar y clasificar vehículos en imágenes o video.

### Tecnologías

- **Modelo:** YOLOv8 (pre-entrenado en COCO dataset)
- **Lenguaje:** Python
- **Librerías:** OpenCV, Ultralytics

### Resultados Esperados

-  Detección de autos (clase "car")
-  Detección de motos (clase "motorcycle")
-  Visualización con bounding boxes y etiquetas de precisión

---

##    Tecnologías Utilizadas

| Práctica | Tecnologías |
|----------|-------------|
| **Práctica 1** | ESP32, C, MicroPhyton, Interrupciones, Wokwi |
| **Práctica 2** | MicroPython, YOLO, OpenCV, TensorFlow, VS Code |

---

##    Cómo Ejecutar

### Práctica 1: Control de LEDs con Pulsadores

1. Conecta los componentes según el diagrama
2. Abre el proyecto en **Wokwi** o en **ESP-IDF**
3. Compila y carga el código en la ESP32
4. Presiona los pulsadores para encender/apagar los LEDs

   
