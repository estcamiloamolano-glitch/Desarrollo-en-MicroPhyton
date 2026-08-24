# Detección de Carros y Motos mediante YOLO y ESP32

## 1. Descripción del proyecto

Este proyecto consiste en el desarrollo de un sistema de visión artificial capaz de detectar **carros y motocicletas en tiempo real mediante una cámara**, utilizando un modelo de inteligencia artificial basado en **YOLO**.

El sistema fue desarrollado en Python y posteriormente integrado con una **ESP32 WROOM**, la cual recibe mediante comunicación serial la información obtenida por el modelo de detección. Dependiendo del objeto identificado, la ESP32 controla dos indicadores LED:

- 🔴 **LED rojo:** indica la detección de un carro.
- 🟢 **LED verde:** indica la detección de una motocicleta.
- 🔴🟢 **Ambos LEDs:** indican que se detectaron simultáneamente un carro y una motocicleta.
- ⚫ **Ningún LED:** indica que no se detectaron objetos de las clases entrenadas.

La comunicación entre el computador y la ESP32 se realiza mediante el cable USB, utilizando comunicación serial.

### Arquitectura general del sistema

```text
                    ┌─────────────────┐
                    │     CÁMARA      │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     PYTHON      │
                    │                 │
                    │     OpenCV      │
                    │       +         │
                    │      YOLO       │
                    └────────┬────────┘
                             │
                     Comunicación
                       Serial USB
                             │
                             ▼
                    ┌─────────────────┐
                    │      ESP32      │
                    │                 │
                    │  GPIO 25 → 🔴   │
                    │  GPIO 26 → 🟢   │
                    └─────────────────┘
