# Seeed Studio XIAO ESP32C3

## 📋 Resumen

El **Seeed Studio XIAO ESP32C3** es un microcontrolador ultra compacto (tamaño de pulgar) con capacidad WiFi y Bluetooth Low Energy (BLE) integrada. Ideal para proyectos IoT, wearables y aplicaciones de baja potencia.

## 📊 Especificaciones Técnicas

| Característica | Valor |
|---|---|
| **Microcontrolador** | ESP32-C3 (RISC-V) |
| **Frecuencia de CPU** | 160 MHz |
| **RAM** | 400 KB SRAM |
| **Flash** | 4 MB (divididos: 384 KB para bootloader/sistema, 3.65 MB disponible para aplicaciones) |
| **GPIO** | 11 pines digitales (E0-E10, excepto E5) |
| **ADC** | 4 canales de 12 bits |
| **PWM** | Todos los pines GPIO pueden ser PWM |
| **UART** | 2 puertos UART (UART0 para programación, UART1 para general) |
| **SPI** | 1 puerto SPI |
| **I2C** | 1 puerto I2C |
| **Conectividad** | WiFi 802.11 b/g/n (2.4 GHz) + BLE 5.0 |
| **Consumo en reposo** | ~5 mA (WiFi ON), ~0.05 mA (Deep Sleep) |
| **Voltaje de operación** | 3.3V (rango: 3.0V - 3.6V) |
| **Dimensiones** | 20 mm × 17 mm × 3.5 mm |
| **Peso** | ~2.5g |
| **Temperatura de operación** | -40°C a +85°C |
| **Certificaciones** | FCC, CE, RoHS |

### Pines de Potencia
- **+5V**: Alimentación USB (entrada)
- **3V3**: Salida 3.3V regulada (~200mA máximo)
- **GND**: Tierra (2 pines)

## 🔌 Diagrama de Pinout

```
                    ┌─────────────────┐
                    │   XIAO ESP32C3  │
                    │                 │
                    │  Dorso (Bottom) │
                    │                 │
                    │   + + + + + +   │
                    │   + + + + + +   │
                    │   + + + + + +   │
                    │   + + + + + +   │
                    └─────────────────┘

FRENTE (Parte con componentes):

    GND    +5V    3V3
     │      │      │
     ◄──────┴──────►
     
    E0(D0)  - GPIO0  - UART0 RX
    E1(D1)  - GPIO1  - UART0 TX
    E2(D2)  - GPIO2  - ADC0
    E3(D3)  - GPIO3  - ADC1
    E4(D4)  - GPIO4  - ADC2
    
    GND ────────────── (Tierra)
    
    E6(D6)  - GPIO6  - UART1 TX
    E7(D7)  - GPIO7  - UART1 RX
    E8(D8)  - GPIO8  - SCL (I2C)
    E9(D9)  - GPIO9  - SDA (I2C)
    E10(D10)- GPIO10 - ADC3

Notas:
- E5 no está disponible (LED RGB integrado)
- Todos los pines son tolerantes a 3.3V
```

## ⚡ Características Principales

### Conectividad
- **WiFi**: Conexión a redes 802.11 b/g/n
- **BLE 5.0**: Bluetooth de bajo consumo para dispositivos móviles
- **Dual Mode**: Puede operar como WiFi + BLE simultáneamente

### Eficiencia Energética
- Modo Deep Sleep reduce el consumo a 0.05 mA
- Perfecto para aplicaciones alimentadas por batería
- Wake-up por GPIO configurable

### Facilidad de Desarrollo
- Compatible con Arduino IDE
- Soporte en PlatformIO
- Bootloader DFU (sin necesidad de botón RESET)
- Depuración JTAG integrada

## 🔧 Configuración de Pines por Función

### UART (Comunicación Serial)
```python
# UART0 (Programación/Debug)
RX: E0 (GPIO 0)
TX: E1 (GPIO 1)
Baud Rate: 115200

# UART1 (Disponible para usuario)
RX: E7 (GPIO 7)
TX: E6 (GPIO 6)
```

### SPI
```python
# SPI por defecto
CS:  E10 (GPIO 10)
CLK: E8  (GPIO 8)
MOSI: E9 (GPIO 9)
MISO: E7 (GPIO 7)
```

### I2C
```python
# I2C por defecto
SCL: E8 (GPIO 8)
SDA: E9 (GPIO 9)
```

## 🎯 Primeros Pasos

### Requisitos
- Cable USB tipo C
- Arduino IDE 1.8.19+ o PlatformIO
- Driver CH340 (si es necesario - ver [guía de drivers](#drivers))

### Instalación en Arduino IDE

1. Abrir Arduino IDE
2. Ir a **Archivo → Preferencias**
3. En URLs de gestor de placas, añadir:
   ```
   https://raw.githubusercontent.com/Seeeduino/seeeduino-xiao-esp32c3/main/package_seeeduino_xiao_esp32c3_index.json
   ```
4. Ir a **Herramientas → Placa → Gestor de placas**
5. Buscar "XIAO ESP32C3" e instalar
6. Seleccionar en **Herramientas → Placa → Seeeduino XIAO ESP32C3**

### Primer Programa: Parpadeo del LED

```cpp
// Conexiones:
// LED (ánodo) → Resistencia 220Ω → E3 (GPIO 3)
// LED (cátodo) → GND

const int LED_PIN = 3; // GPIO3

void setup() {
  pinMode(LED_PIN, OUTPUT);
  Serial.begin(115200);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  Serial.println("LED ON");
  delay(1000);
  
  digitalWrite(LED_PIN, LOW);
  Serial.println("LED OFF");
  delay(1000);
}
```

## 📡 Ejemplo WiFi + BLE

```cpp
#include <WiFi.h>
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEServer.h>

const char* ssid = "Tu_SSID";
const char* password = "Tu_Contraseña";

void setup() {
  Serial.begin(115200);
  delay(100);

  // Iniciar WiFi
  WiFi.begin(ssid, password);
  int attempt = 0;
  while (WiFi.status() != WL_CONNECTED && attempt < 20) {
    delay(500);
    Serial.print(".");
    attempt++;
  }
  
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nWiFi conectado");
    Serial.println(WiFi.localIP());
  }

  // Iniciar BLE
  BLEDevice::init("XIAO-ESP32C3");
  BLEServer *pServer = BLEDevice::createServer();
  BLEService *pService = pServer->createService("180A");
  BLECharacteristic * pCharacteristic = pService->createCharacteristic(
                                         "2A29",
                                         BLECharacteristic::PROPERTY_READ
                                       );
  pCharacteristic->setValue("Seeeduino XIAO ESP32C3");
  pService->start();
  
  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->addServiceUUID("180A");
  pAdvertising->setScanResponse(true);
  pAdvertising->setMinPreferred(0x06);
  pAdvertising->setMinPreferred(0x12);
  BLEDevice::startAdvertising();
  
  Serial.println("BLE iniciado - esperando conexiones...");
}

void loop() {
  delay(2000);
  Serial.print("WiFi IP: ");
  Serial.println(WiFi.localIP());
}
```

## 🐛 Solución de Problemas

### No se detecta el puerto COM/USB

!!! warning "Puerto no reconocido"
    1. Instalar driver CH340 desde [sitio oficial](https://wiki.seeedstudio.com/XIAO_ESP32C3_Getting_Started/)
    2. En Linux: `sudo usermod -a -G dialout $USER` (reiniciar sesión)
    3. Verificar conexión USB con otro cable

### Upload fallido

```
esptool.py v4.x.x
Serial port /dev/ttyUSBX
Connecting....
```

Si se queda aquí, presionar Ctrl+C y:

1. Mantener presionado BOOT (botón derecho) durante 2-3 segundos
2. Presionar RESET (botón izquierdo)
3. Intentar upload nuevamente

### Bajo rendimiento WiFi

- Alejar de redes WiFi de 5GHz (el XIAO solo soporta 2.4GHz)
- Aumentar la potencia TX: `WiFi.setTxPower(WIFI_POWER_19_5dBm);`
- Reducir interferencias: cambiar canal WiFi

## 📚 Recursos Adicionales

- [Wiki Oficial Seeeduino XIAO ESP32C3](https://wiki.seeedstudio.com/XIAO_ESP32C3_Getting_Started/)
- [Datasheet ESP32-C3](https://www.espressif.com/en/products/socs/esp32-c3)
- [Arduino Core para ESP32](https://github.com/espressif/arduino-esp32)
- [Ejemplos en GitHub](https://github.com/Seeeduino/seeeduino-xiao-esp32c3)

## 📝 Revisiones de Hardware

| Revisión | Fecha | Cambios |
|---|---|---|
| v1.0 | Enero 2022 | Versión inicial |
| v1.1 | Marzo 2023 | Mejoras en regulador de voltaje |
| v1.2 | Octubre 2024 | Optimización de antena BLE (actual) |

---

**Última actualización**: 9 de febrero de 2026  
**Documentación versión**: 2.1
