# Preguntas Frecuentes (FAQ)

## General

### ¿Cuál es la diferencia entre XIAO ESP32C3 y ESP32 estándar?

| Característica | XIAO ESP32C3 | ESP32 Estándar |
|---|---|---|
| Tamaño | 20×17 mm (pulgar) | 50×23 mm |
| Núcleo | RISC-V single-core | Xtensa dual-core |
| GPIO | 11 | 34 |
| RAM | 400 KB | 520 KB |
| Precio | $ ~ 13 USD | $ ~ 8 USD |
| Ideal para | Wearables, Proyectos compactos | Aplicaciones grandes |

### ¿Puedo usar ambas redes (WiFi + BLE) simultáneamente?

Sí, el XIAO ESP32C3 soporta coexistencia WiFi + BLE. Ejemplo:

```cpp
#include <WiFi.h>
#include <BLEDevice.h>

void setup() {
  WiFi.begin("SSID", "password");
  BLEDevice::init("XIAO-IoT");
}
```

## Alimentación y Consumo

### ¿Cuál es el consumo de corriente?

- **WiFi activo**: ~80-160 mA
- **BLE activo**: ~10-25 mA  
- **Reposo (WiFi OFF)**: ~5 mA
- **Deep Sleep**: ~0.05 mA

### ¿Puedo alimentarlo con batería?

Sí. Para aplicaciones con batería:

```cpp
// Entrar en Deep Sleep y despertar cada 10 segundos
esp_sleep_enable_timer_wakeup(10 * 1000000);
esp_deep_sleep_start();
```

Estimación con batería 850mAh en Deep Sleep:
- ~17000 horas (1.9 años)

### ¿Qué voltaje máximo soporta el pin de entrada analógica?

Los pines ADC soportan máximo **3.3V**. Si tienes una señal mayor, usar un divisor de voltaje:

```
Vin ──────[R1=10kΩ]──────┬──► GPIO (0-3.3V)
                         │
                       [R2=10kΩ]
                         │
                        GND
```

## Programación

### ¿Cuál es la velocidad de comunicación serial recomendada?

**115200 baud** es el estándar y lo más rápido sin problemas de paridad. 
Para velocidades mayores (230400, 460800), usar cables USB de calidad cortos (<2m).

### ¿Cómo resetear el XIAO sin desconectar el USB?

```cpp
void setup() {
  Serial.begin(115200);
  Serial.println("Sistema iniciado");
}

void loop() {
  if (Serial.available()) {
    char cmd = Serial.read();
    if (cmd == 'r' || cmd == 'R') {
      Serial.println("Reiniciando...");
      delay(100);
      ESP.restart();  // ← Reset por software
    }
  }
}
```

### ¿Puedo usar interrupciones en múltiples pines?

Sí, pero con cuidado. Máximo 10 interrupciones simultáneas:

```cpp
const int PIN1 = 2;
const int PIN2 = 3;

volatile int contador1 = 0;
volatile int contador2 = 0;

void ISR1() { contador1++; }
void ISR2() { contador2++; }

void setup() {
  attachInterrupt(digitalPinToInterrupt(PIN1), ISR1, RISING);
  attachInterrupt(digitalPinToInterrupt(PIN2), ISR2, RISING);
}
```

## Conectividad

### ¿Cómo conectarme a WiFi con contraseña WPA2?

```cpp
#include <WiFi.h>

void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  WiFi.begin("NombreSSID", "Contraseña");
  
  int attempts = 0;
  while (WiFi.status() != WL_CONNECTED && attempts < 20) {
    delay(500);
    Serial.print(".");
    attempts++;
  }
  
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nConectado!");
    Serial.println(WiFi.localIP());
  }
}
```

### ¿Cómo crear un punto de acceso (AP) con el XIAO?

```cpp
#include <WiFi.h>

void setup() {
  Serial.begin(115200);
  
  // Crear red WiFi abierta
  WiFi.softAP("XIAO-AP", "");  // SSID y contraseña
  IPAddress IP = WiFi.softAPIP();
  Serial.print("IP del AP: ");
  Serial.println(IP);
}
```

### ¿Cuál es el rango máximo de BLE?

- **Línea de vista**: ~100 metros
- **Con obstáculos**: ~10-30 metros
- Depende de la potencia TX y antena receptora

### ¿Puedo usar HTTPS seguro?

Sí, usando `WiFiClientSecure`:

```cpp
#include <WiFiClientSecure.h>

const char* ssid = "Tu_SSID";
const char* password = "Tu_Pass";
const char* host = "api.example.com";
const int httpsPort = 443;

// Certificado raíz (actualizar según necesidad)
const char* root_ca = R"EOF(
-----BEGIN CERTIFICATE-----
... (contenido certificado)
-----END CERTIFICATE-----
)EOF";

void setup() {
  WiFi.begin(ssid, password);
  
  WiFiClientSecure client;
  client.setCACert(root_ca);
  
  if (!client.connect(host, httpsPort)) {
    Serial.println("Conexión fallida");
    return;
  }
  
  client.print(String("GET /path HTTP/1.1\r\n") +
               "Host: " + host + "\r\n" +
               "Connection: close\r\n\r\n");
}
```

## Hardware

### ¿Qué pines puedo usar para PWM?

Todos los GPIO excepto E5 (LED RGB). Para máxima precisión:

```cpp
const int PWM_PIN = 3;      // Cualquier GPIO
const int PWM_FREQ = 1000;  // Hz
const int PWM_RES = 8;      // bits (0-255)

void setup() {
  ledcSetup(0, PWM_FREQ, PWM_RES);
  ledcAttachPin(PWM_PIN, 0);
}

void loop() {
  for(int brightness = 0; brightness < 256; brightness++) {
    ledcWrite(0, brightness);
    delay(10);
  }
}
```

### ¿Cómo usar ADC para medir voltaje?

```cpp
const int ADC_PIN = A0;  // Ej: E2 (GPIO2)

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);  // 12 bits (0-4095)
}

void loop() {
  int rawValue = analogRead(ADC_PIN);
  float voltage = (rawValue / 4095.0) * 3.3;
  
  Serial.print("Raw: ");
  Serial.print(rawValue);
  Serial.print(" | Voltaje: ");
  Serial.print(voltage, 3);
  Serial.println("V");
  
  delay(500);
}
```

### ¿Cómo evitar ruido en lecturas ADC?

```cpp
// Promedio de múltiples lecturas
int readADCFiltered(int pin, int samples = 10) {
  long sum = 0;
  for(int i = 0; i < samples; i++) {
    sum += analogRead(pin);
    delayMicroseconds(100);
  }
  return sum / samples;
}

void loop() {
  int valor = readADCFiltered(A0, 20);
  Serial.println(valor);
}
```

## Almacenamiento de Datos

### ¿Cómo guardar datos en flash?

```cpp
#include <SPIFFS.h>

void setup() {
  if (!SPIFFS.begin(true)) {
    Serial.println("Error al montar SPIFFS");
    return;
  }
  
  // Escribir archivo
  File file = SPIFFS.open("/data.txt", "w");
  file.println("Hola, mundo!");
  file.close();
  
  // Leer archivo
  file = SPIFFS.open("/data.txt", "r");
  while (file.available()) {
    Serial.write(file.read());
  }
  file.close();
}
```

### ¿Cuánto espacio flash está disponible para aplicaciones?

Total: 4 MB
- Bootloader: 384 KB
- Partición SPIFFS: ~384 KB
- **Disponible para código**: ~3.2 MB

## Temperatura y Ambiente

### ¿Cuál es el rango de temperatura de operación?

**-40°C a +85°C** (rango industrial)

Para lectura de temperatura interna:

```cpp
void setup() {
  Serial.begin(115200);
}

void loop() {
  float temp = (float) temperatureRead();
  Serial.print("Temperatura interna: ");
  Serial.print(temp, 1);
  Serial.println("°C");
  delay(1000);
}
```

### ¿Es resistente a la humedad?

El XIAO no tiene revestimiento conformal. Para ambientes húmedos:
- Aplicar acrílico claro protector
- Utilizar carcasas IP67 con drenaje de aire
- Evitar condensación (secador al encender)

## Actualizaciones y Firmware

### ¿Cómo actualizar el bootloader?

El bootloader DFU se actualiza automáticamente. Para forzar:

```bash
# Descargar herramienta esptool
pip install esptool

# Entrar en modo DFU (mantener BOOT + RESET)
esptool.py --chip esp32c3 --port /dev/ttyUSBX erase_flash
```

### ¿Puedo reclamar la flash usada por SPIFFS?

Sí, reparticionando en Arduino IDE:

1. **Herramientas → Partición** → Seleccionar tamaño SPIFFS deseado
2. **Herramientas → Borrar flash**
3. Recargar programa

---

**Última actualización**: 9 de febrero de 2026  
**Preguntas**: ¿Tienes una pregunta no incluida? Crear un [issue en GitHub](https://github.com/tu-empresa/documentacion-publica/issues)
