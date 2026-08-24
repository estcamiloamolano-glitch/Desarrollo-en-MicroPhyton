# Práctica 2: Detección de Carros y Motos mediante YOLO e Integración con ESP32

## 1. Descripción

Esta práctica consiste en el desarrollo de un sistema de visión artificial capaz de detectar carros y motocicletas en tiempo real mediante una cámara, utilizando un modelo de inteligencia artificial basado en YOLO (You Only Look Once).

El sistema fue desarrollado en Python y posteriormente integrado con una ESP32 WROOM, la cual recibe mediante comunicación serial la información obtenida por el modelo de detección. Dependiendo del objeto identificado, la ESP32 controla dos indicadores LED:

- **LED rojo (GPIO 25):** indica la detección de un carro.
- **LED verde (GPIO 26):** indica la detección de una motocicleta.
- **Ambos LEDs encendidos:** indican que se detectaron simultáneamente un carro y una motocicleta.
- **Ningún LED encendido:** indica que no se detectaron objetos de las clases entrenadas.

La comunicación entre el computador y la ESP32 se realiza mediante el cable USB, utilizando comunicación serial.

---

## 2. Arquitectura General del Sistema
