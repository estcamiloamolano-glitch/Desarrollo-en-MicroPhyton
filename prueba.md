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
│ GPIO 25 → 🔴 LED │
│ GPIO 26 → 🟢 LED │
└─────────────────┘
```


---

## 3. Tecnologías Utilizadas

- Python 3.12.10
- Visual Studio Code
- YOLO (You Only Look Once)
- Ultralytics
- OpenCV
- PySerial
- PyTorch
- ESP32 WROOM
- Arduino IDE
- Cámara web
- LEDs rojo y verde
- Resistencias para los LEDs
- Comunicación serial mediante USB

Durante el entrenamiento se trabajó con CPU, sin utilizar CUDA ni una GPU dedicada.

---

## 4. Estructura del Proyecto

```text
DETECCIÓN DE CARROS Y MOTOS/
│
├── Dataset/ # Conjunto de datos para entrenamiento
│
├── Motos nuevas/ # Imágenes de motocicletas recopiladas
│
├── Motos nuevas JPG/ # Imágenes convertidas a formato JPG
│
├── motos_exportadas/ # Dataset exportado con anotaciones
│ ├── labels/ # Etiquetas en formato YOLO
│ ├── data.yaml # Configuración del dataset
│ └── train.txt # Lista de archivos de entrenamiento
│
├── runs/ # Resultados del entrenamiento
│ └── detect/
│ └── motos_mejorado/
│ └── weights/
│ ├── best.pt # Mejor modelo obtenido
│ └── last.pt # Último estado del entrenamiento
│
├── venv/ # Entorno virtual de Python
│
├── agregar_motos.py # Preparación del dataset de motos
├── camera.py # Programa principal de detección
├── convertir_jpg.py # Conversión de imágenes a JPG
├── moto_prueba.jpg # Imagen de prueba
├── probar_modelo.py # Pruebas individuales
├── resultado_moto.jpg # Resultado de detección
├── test_model.py # Pruebas del modelo entrenado
├── train.py # Entrenamiento del modelo
└── yolo26n.pt # Modelo base para transfer learning
```


---

## 5. Creación del Entorno Virtual

Para evitar conflictos entre las bibliotecas del proyecto y las instaladas globalmente en el sistema, se creó un entorno virtual llamado `venv`.

Desde la terminal de Visual Studio Code se ejecutó:

```powershell
python -m venv venv
```
---
Posteriormente se activó el entorno virtual mediante:
```powershell
venv\Scripts\activate
```
---
Una vez activado correctamente, la terminal muestra (venv) al comienzo de la línea de comandos.
---
## 6. Instalación de Bibliotecas
Dentro del entorno virtual se instalaron las siguientes bibliotecas:

```powershell
pip install ultralytics      # Para trabajar con YOLO
pip install opencv-python    # Para acceso a cámara y procesamiento de imágenes
pip install pyserial         # Para comunicación serial con ESP32
```
---
-pip install ultralytics      # Para trabajar con YOLO
-pip install opencv-python    # Para acceso a cámara y procesamiento de imágenes
-pip install pyserial         # Para comunicación serial con ESP32

---
## 7. Preparación del Conjunto de Datos
Para entrenar el detector se preparó un conjunto de imágenes que contiene dos clases:

```powershell
Clase 0 → Cars
Clase 1 → Motorcycle
```

El objetivo fue que el modelo pudiera diferenciar entre carros y motocicletas.

Las imágenes fueron organizadas en conjuntos de:

Entrenamiento `(train)`

Validación `(valid)`

Prueba `(test)`

La estructura utilizada para el conjunto de datos fue:

```text
dataset/
│
├── train/
│   ├── images/
│   └── labels/
│
├── valid/
│   ├── images/
│   └── labels/
│
└── test/
    ├── images/
    └── labels/
```
---
## 8. Distribución del Dataset.
Durante la preparación del conjunto de datos se verificó la correspondencia entre imágenes y etiquetas:
  
  ```powershell
  TRAIN:   203 imágenes, 203 etiquetas
  VALID:   27 imágenes,  27 etiquetas
  TEST:    20 imágenes,  20 etiquetas
  ```

La cantidad de imágenes y etiquetas coincidió en cada conjunto, permitiendo realizar correctamente el entrenamiento.

---
## 9. Archivo data.yaml
El archivo data.yaml es fundamental para el entrenamiento, ya que le indica a YOLO dónde encontrar las imágenes y etiquetas correspondientes:

  ```powershell
  train: train/images
  val: valid/images
  test: test/images

  nc: 2

  names:
    0: Cars
    1: Motorcycle

  ```
---
Donde:
---
- `train` corresponde a las imágenes utilizadas para entrenar.

- `val` corresponde al conjunto de validación.

- `test` corresponde al conjunto utilizado para realizar pruebas.

- `nc: 2` indica que existen dos clases.

- `Cars` representa los carros.

- `Motorcycle` representa las motocicletas.

---

## 10. Modelo Inicial
Para comenzar el proyecto se utilizó un modelo YOLO previamente disponible como punto de partida:

```text
yolo26n.pt
```

Este modelo fue utilizado como base para el entrenamiento mediante transfer learning, permitiendo adaptarlo a las dos clases específicas requeridas para el proyecto.

---
## 11. Entrenamiento del Modelo
El entrenamiento se realizó mediante Python utilizando la biblioteca Ultralytics. El objetivo fue adaptar YOLO para reconocer específicamente carros y motocicletas.

El entrenamiento se ejecutó utilizando la CPU del computador y tuvo una duración aproximada de 3.615 horas, completando 38 épocas.

Durante el entrenamiento se utilizó un mecanismo de parada temprana (EarlyStopping) para evitar continuar entrenando cuando el modelo dejaba de presentar mejoras significativas. El mejor resultado se obtuvo alrededor de la época 23.

Al finalizar el entrenamiento se generaron los siguientes archivos:

```text
runs/
└── detect/
    └── motos_mejorado/
        └── weights/
            ├── best.pt    # Mejor modelo obtenido
            └── last.pt    # Último estado del entrenamiento
```
El archivo best.pt corresponde al mejor modelo obtenido durante el entrenamiento y es el que posteriormente se utiliza para realizar las detecciones.

---
## 12. Pruebas del Modelo
Antes de utilizar la cámara en tiempo real, se realizaron pruebas utilizando imágenes mediante los programas:

- test_model.py: Para pruebas del modelo sobre el conjunto de evaluación

- probar_modelo.py: Para pruebas individuales de detección

Estas pruebas permitieron comprobar que el modelo podía detectar correctamente las clases Cars y Motorcycle, incluso en imágenes que contenían simultáneamente ambos objetos.

## 13. Detección mediante Cámara
Una vez comprobado el funcionamiento del modelo, se implementó la detección en tiempo real mediante una cámara web. El programa principal utilizado para esta etapa es `camera.py`.

El modelo entrenado se carga mediante:

```python
model = YOLO(r"runs\detect\runs\detect\motos_mejorado\weights\best.pt")
```
---
Posteriormente se abre la cámara mediante OpenCV con una resolución de 640x480 píxeles:

```python
cap = cv2.VideoCapture(0)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
```
---
Para las detecciones se estableció un nivel de confianza de 0.40 (40%):

```python
results = model(frame, conf=0.40, verbose=False)
```
---
## 14. Conteo de Objetos Detectados
Después de ejecutar YOLO sobre cada imagen, el programa revisa las cajas detectadas. Para cada detección se obtiene la clase:

```python
class_id = int(box.cls[0])
```
La clase se interpreta de la siguiente manera:

```text
0 → Carro
1 → Motocicleta
```

```python
if class_id == 0:
    cars += 1
elif class_id == 1:
    motorcycles += 1
```
El programa puede mostrar en pantalla el conteo de objetos detectados:

```text
CARROS: 1
MOTOS: 0
```
Las detecciones son dibujadas directamente sobre la imagen mediante:

```python
annotated_frame = results[0].plot()
```
---

## 15. Integración con la ESP32
Después de comprobar la detección mediante cámara, se integró el sistema con una ESP32 WROOM. La ESP32 funciona como el dispositivo encargado de controlar los indicadores físicos.

La conexión utilizada fue:

```text
GPIO 25 → LED rojo
GPIO 26 → LED verde
```
Los LEDs deben conectarse utilizando una resistencia limitadora de corriente. La comunicación entre Python y la ESP32 se realiza mediante el puerto USB, sin necesidad de utilizar Wi-Fi o Bluetooth.

---
## 16. Programa de la ESP32
La ESP32 fue programada mediante Arduino IDE. Se configuraron los pines como salidas:

```cpp
const int LED_ROJO = 25;
const int LED_VERDE = 26;

void setup() {
    pinMode(LED_ROJO, OUTPUT);
    pinMode(LED_VERDE, OUTPUT);
    Serial.begin(115200);
}
```
---
## 17. Comandos de Comunicación
Para comunicar la información de YOLO a la ESP32 se establecieron cuatro comandos:

```text
Comando	Significado	LED rojo	LED verde
N	Ninguno	OFF	OFF
C	Carro	ON	OFF
M	Moto	OFF	ON
B	Ambos	ON	ON
```
La ESP32 recibe estos caracteres mediante el puerto serial y modifica el estado de los GPIO:

```cpp
void loop() {
    if (Serial.available() > 0) {
        char comando = Serial.read();
        
        switch(comando) {
            case 'N':
                digitalWrite(LED_ROJO, LOW);
                digitalWrite(LED_VERDE, LOW);
                break;
            case 'C':
                digitalWrite(LED_ROJO, HIGH);
                digitalWrite(LED_VERDE, LOW);
                break;
            case 'M':
                digitalWrite(LED_ROJO, LOW);
                digitalWrite(LED_VERDE, HIGH);
                break;
            case 'B':
                digitalWrite(LED_ROJO, HIGH);
                digitalWrite(LED_VERDE, HIGH);
                break;
        }
    }
}
```
---

## 18. Comunicación desde Python
Para realizar la comunicación serial desde Python se utilizó la biblioteca PySerial:

```python
import serial

ESP32_PORT = "COM3"  # Verificar el puerto correcto
BAUDRATE = 115200

esp32 = serial.Serial(ESP32_PORT, BAUDRATE, timeout=1)
```

El puerto COM puede variar dependiendo del computador y de la conexión USB. Se debe verificar en Arduino IDE: Herramientas → Puerto.
---

## 19. Lógica de Integración
La lógica final del sistema es la siguiente:

```text
                CÁMARA
                   │
                   ▼
             Imagen en vivo
                   │
                   ▼
                 YOLO
                   │
          ┌────────┼────────┐
          │        │        │
          ▼        ▼        ▼
       Carro      Moto    Ninguno
          │        │        │
          C        M        N
          │        │        │
          └────────┼────────┘
                   │
                   ▼
              Comunicación
                 Serial
                   │
                   ▼
                 ESP32
                   │
          ┌────────┴────────┐
          ▼                 ▼
       GPIO 25           GPIO 26
          │                 │
          ▼                 ▼
       LED ROJO          LED VERDE

```
Cuando aparecen simultáneamente un carro y una motocicleta, el programa envía el comando 'B' y ambos indicadores se encienden.

---
## 20. Optimización de la Comunicación
Durante la integración se implementó una variable para almacenar el último comando enviado:

```python
ultimo_comando = ""
```
Esto evita enviar el mismo comando continuamente en cada fotograma. Por ejemplo, si YOLO detecta un carro durante varios segundos, no es necesario enviar 'C' en cada fotograma. En cambio, Python envía 'C' solo cuando cambia el estado:

```text
N → C  (aparece un carro)
C → N  (desaparece el carro)
N → M  (aparece una moto)
Esto reduce la cantidad de información transmitida por el puerto serial y hace más eficiente la comunicación.
```

## 21. Ejecución del Sistema Completo
Para utilizar el sistema completo se deben seguir los siguientes pasos:

*Paso 1. Conectar la ESP32*
Conectar la ESP32 al computador mediante USB.

*Paso 2. Programar la ESP32*
Abrir el proyecto en Arduino IDE y cargar el programa correspondiente a la ESP32.

*Paso 3. Verificar el puerto COM*
Identificar el puerto asignado a la ESP32 (ejemplo: COM3).

*Paso 4. Cerrar el Monitor Serial*
Antes de ejecutar Python se debe cerrar el Monitor Serial de Arduino IDE, ya que Python necesita utilizar el mismo puerto COM.

*Paso 5. Activar el entorno virtual*
Desde la terminal de Visual Studio Code:

```powershell
venv\Scripts\activate
```
*Paso 6. Ejecutar* `camera.py`

```powershell
python camera.py
```
*Paso 7. Realizar las pruebas*
Colocar frente a la cámara:

- Un carro

- Una motocicleta

- Ambos objetos

- Ningún objeto

---
## 22. Comportamiento Esperado
El comportamiento esperado del sistema es el siguiente:

```text
CARRO         → LED ROJO encendido
MOTO          → LED VERDE encendido
CARRO + MOTO  → LED ROJO + LED VERDE encendidos
NINGUNO       → LEDs apagados
```

---
## 23. Tabla de Archivos Principales

| Archivo | Función |
|:--------|:--------|
| `camera.py` | Detección en tiempo real y comunicación con ESP32 |
| `train.py` | Entrenamiento del modelo |
| `test_model.py` | Pruebas del modelo |
| `probar_modelo.py` | Pruebas individuales de detección |
| `agregar_motos.py` | Preparación/ampliación de imágenes de motocicletas |
| `convertir_jpg.py` | Conversión de imágenes a JPG |
| `data.yaml` | Configuración del conjunto de datos |
| `best.pt` | Mejor modelo obtenido durante el entrenamiento |
| `last.pt` | Último estado del entrenamiento |
| `yolo26n.pt` | Modelo utilizado como base |

---
## 24. Conclusión
El proyecto permitió desarrollar e integrar un sistema de detección de carros y motocicletas utilizando técnicas de visión artificial con YOLO.

Inicialmente se preparó el conjunto de datos y se organizaron las imágenes y etiquetas correspondientes a las dos clases. Posteriormente se configuró un entorno virtual de Python y se instalaron las bibliotecas necesarias para el entrenamiento y procesamiento de imágenes.

Después se realizó el entrenamiento del modelo, obteniendo como resultado el archivo best.pt, que posteriormente fue utilizado para realizar detecciones mediante una cámara web.

Finalmente, el modelo de visión artificial se integró con una ESP32 WROOM mediante comunicación serial USB. La información obtenida por YOLO se transformó en comandos que permiten controlar dos LEDs conectados a los GPIO 25 y 26.

El resultado final permite detectar carros y motocicletas en tiempo real y representar físicamente cada detección mediante indicadores luminosos, demostrando la integración entre inteligencia artificial, visión artificial, programación y sistemas embebidos.

---
## 25. Requisitos del Sistema
- Python 3.12 o superior

- Entorno virtual (venv)

- Cámara web

- ESP32 con programa cargado

- Puertos USB disponibles

---
## 26. Licencia
Este proyecto está bajo la Licencia MIT.
---
## 27. Autor
Camilo Andrés Molano Umaña
GitHub
