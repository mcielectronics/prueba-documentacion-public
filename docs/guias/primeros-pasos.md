# Guía de Primeros Pasos

## 🎯 Objetivo

Esta guía te mostrará cómo configurar tu entorno de desarrollo y ejecutar tu primer programa en el **Seeed Studio XIAO ESP32C3** en menos de 10 minutos.

## 📋 Requisitos

- [ ] Seeed Studio XIAO ESP32C3
- [ ] Cable USB tipo C (incluido)
- [ ] Computadora con Windows, macOS o Linux
- [ ] Conexión a Internet
- [ ] (Opcional) Protoboard y algunos LEDs/resistencias para experimentos

## 🔧 Paso 1: Instalar Arduino IDE

### Windows y macOS
1. Descargar desde [arduino.cc/downloads](https://www.arduino.cc/software)
2. Ejecutar el instalador
3. Completar la instalación por defecto

### Linux (Ubuntu/Debian)
```bash
sudo apt-get install arduino
```

O usando Snap:
```bash
sudo snap install arduino
```

!!! tip "Versión mínima"
    Se requiere Arduino IDE **1.8.19** o superior. Verificar en **Ayuda → Acerca de Arduino**.

## 🔌 Paso 2: Instalar el Gestor de Placas

### Configurar URL del Gestor de Placas

1. Abrir Arduino IDE
2. Ir a **Archivo → Preferencias** (o **Arduino → Settings** en macOS)
3. Encontrar el campo **URLs adicionales de gestor de placas**
4. Copiar y pegar la siguiente URL:
   ```
   https://raw.githubusercontent.com/Seeeduino/seeeduino-xiao-esp32c3/main/package_seeeduino_xiao_esp32c3_index.json
   ```
5. Hacer clic en **OK**

### Instalar la Placa ESP32C3

1. Ir a **Herramientas → Placa → Gestor de placas**
2. Buscar: `XIAO ESP32C3`
3. Hacer clic en el resultado
4. Seleccionar la versión más reciente
5. Hacer clic en **Instalar**
   
⏳ La instalación puede tomar 2-3 minutos (descarga ~150 MB)

## 🔌 Paso 3: Conectar el Hardware

1. Conectar el XIAO ESP32C3 a la computadora mediante cable USB tipo C
2. El LED RGB integrado parpadeará en rojo (indicando modo bootloader)
3. Abrimos Arduino IDE

### Seleccionar Puerto y Placa

1. **Herramientas → Placa** → Seleccionar **Seeeduino XIAO ESP32C3**
2. **Herramientas → Puerto** → Seleccionar el puerto COM/ttyUSB que aparezca
   - Windows: `COM3`, `COM4`, etc.
   - macOS: `/dev/cu.usbserial-*`
   - Linux: `/dev/ttyUSB0`, `/dev/ttyUSB1`, etc.

!!! info "Ayuda para identificar puerto"
    1. Desconectar el XIAO
    2. Abrir Herramientas → Puerto y anotar qué puertos hay
    3. Conectar el XIAO
    4. El nuevo puerto que aparece es el correcto

## 📝 Paso 4: Crear tu Primer Programa

### Código: Parpadeo LED Integrado

```cpp
// Seeed Studio XIAO ESP32C3 - Blink LED
// Este ejemplo parpadea el LED integrado cada segundo

const int LED_PIN = 20;  // LED RGB integrado

void setup() {
  // Inicializar puerto serial para debugging
  Serial.begin(115200);
  
  // Configurar pin del LED como salida
  pinMode(LED_PIN, OUTPUT);
  
  Serial.println("Sistema iniciado!");
  Serial.println("LED parpadeando...");
}

void loop() {
  // Encender LED
  digitalWrite(LED_PIN, HIGH);
  Serial.println("LED: ON");
  delay(1000);  // Esperar 1 segundo
  
  // Apagar LED
  digitalWrite(LED_PIN, LOW);
  Serial.println("LED: OFF");
  delay(1000);  // Esperar 1 segundo
}
```

### Cargar el Programa

1. Copiar el código anterior en Arduino IDE
2. Hacer clic en ✓ (Verificar) para compilar
3. Hacer clic en → (Subir) para cargar en el dispositivo
4. Esperar hasta ver: `Leaving... Hard resetting via RTS pin...`

🎉 ¡Tu XIAO ESP32C3 está parpadeando!

## 🖥️ Paso 5: Monitor Serial

Para ver mensajes de la placa:

1. Hacer clic en **Herramientas → Monitor Serial**
2. Asegurarse que la velocidad esté en **115200 baud**
3. Deberías ver:
   ```
   Sistema iniciado!
   LED parpadeando...
   LED: ON
   LED: OFF
   LED: ON
   LED: OFF
   ...
   ```

## 🔌 Paso 6: Experimento Interactivo (Opcional)

### Control LED Externo por Serial

```cpp
// Control de LED por comandos serial
// Enviar "on" para encender, "off" para apagar

const int LED_PIN = 3;  // GPIO3 (pin E3)

void setup() {
  Serial.begin(115200);
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);
  
  Serial.println("=== Control LED por Serial ===");
  Serial.println("Comandos:");
  Serial.println("  'on'  - Encender LED");
  Serial.println("  'off' - Apagar LED");
  Serial.println("  '?'   - Estado actual");
}

void loop() {
  if (Serial.available() > 0) {
    String comando = Serial.readStringUntil('\n');
    comando.trim();
    
    if (comando == "on") {
      digitalWrite(LED_PIN, HIGH);
      Serial.println("✓ LED encendido");
    }
    else if (comando == "off") {
      digitalWrite(LED_PIN, LOW);
      Serial.println("✓ LED apagado");
    }
    else if (comando == "?") {
      int estado = digitalRead(LED_PIN);
      Serial.print("Estado LED: ");
      Serial.println(estado ? "ON" : "OFF");
    }
    else {
      Serial.println("✗ Comando no reconocido");
    }
  }
}
```

### Hardware Necesario:
```
XIAO ESP32C3          Resistencia       LED
─────────────────
    E3 ─────────────[220Ω]─────────┬──► (ánodo)
                                    │
    GND ───────────────────────────┴──► (cátodo)
```

**Pasos:**
1. Conectar el circuito según el esquema
2. Cargar el código anterior
3. Abrir Monitor Serial
4. Escribir comandos: `on`, `off`, `?`

## 🐛 Solución de Problemas

### "Puerto COM no disponible"

```bash
# En Linux, añadir usuario al grupo dialout:
sudo usermod -a -G dialout $USER

# Reiniciar sesión o usar:
sudo usermod -g dialout $USER
newgrp dialout
```

### "No se puede cargar el programa"

**Solución 1: Bootloader DFU**
1. Mientras se compila, mantener presionado el botón **BOOT** (derecha)
2. Presionar brevemente **RESET** (izquierda)
3. Soltar **BOOT**
4. Debería cargarse automáticamente

**Solución 2: Puente de erupción**
```cpp
// En setup(), esperar a que Monitor Serial se abra:
while(!Serial) { delay(100); }
```

### "Arduino IDE no detecta la placa"

1. Reiniciar Arduino IDE
2. Desconectar y reconectar el cable USB
3. Verificar que gestor de placas está instalado
4. En Herramientas → Placa, verificar que **Seeeduino XIAO ESP32C3** está seleccionado

## 📚 Siguientes Pasos

Una vez completada esta guía:

- [ ] Explorar ejemplos adicionales en **Archivo → Ejemplos → Seeed XIAO ESP32C3**
- [ ] Leer la guía de [Especificaciones Técnicas](../productos/xiao-esp32c3.md)
- [ ] Experimentar con ADC (entradas analógicas)
- [ ] Conectarse a WiFi (ver ejemplos en documentación)
- [ ] Usar BLE (Bluetooth Low Energy)

## 💡 Consejos Profesionales

=== "Debugging"

    ```cpp
    // Macro útil para debugging
    #define DEBUG_PRINT(x) Serial.println(#x": " + String(x))
    
    void loop() {
      int sensor = analogRead(A0);
      DEBUG_PRINT(sensor);  // Imprime: sensor: 523
    }
    ```

=== "Bajo Consumo"

    ```cpp
    // Entrar en Deep Sleep por 5 segundos
    esp_sleep_enable_timer_wakeup(5 * 1000000);  // 5 segundos
    esp_deep_sleep_start();  // La ejecución reanuda desde setup()
    ```

=== "Parpadeo sin delay()"

    ```cpp
    unsigned long ultimoTime = 0;
    const unsigned long INTERVALO = 1000;
    
    void loop() {
      if (millis() - ultimoTime >= INTERVALO) {
        digitalWrite(LED_PIN, !digitalRead(LED_PIN));
        ultimoTime = millis();
      }
      // Aquí puedes hacer otras tareas sin bloqueos
    }
    ```

## 📞 Obtener Ayuda

| Recurso | Enlace |
|---|---|
| Wiki Oficial | https://wiki.seeedstudio.com/XIAO_ESP32C3_Getting_Started/ |
| Forum Seeed | https://forum.seeedstudio.com/ |
| Issues GitHub | https://github.com/Seeeduino/seeeduino-xiao-esp32c3/issues |
| Arduino Docs | https://docs.arduino.cc/ |

---

**Última actualización**: 9 de febrero de 2026  
**Tiempo estimado**: 15-20 minutos
