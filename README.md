# Control de Botón con Interrupción en ESP32 (Arduino)

Este repositorio contiene un ejemplo sencillo pero fundamental para el manejo de un botón pulsador en un microcontrolador ESP32 (o similar) utilizando el framework de Arduino. El código demuestra cómo usar una interrupción por hardware para detectar de manera eficiente las pulsaciones de un botón, incluso mientras el programa principal realiza otras tareas.

## 📋 Descripción del Código

El programa configura un botón en el pin GPIO 18 con una resistencia de `pull-up` interna. Se utiliza una interrupción que se dispara en el **flanco de bajada** (`FALLING`) del pin, es decir, cuando el botón es presionado.

- **Interrupción (`isr`):** Cada vez que se presiona el botón, se ejecuta la función `isr`. Esta función, declarada en la RAM (`IRAM_ATTR`) para una ejecución rápida, incrementa un contador (`numberKeyPresses`) y establece una bandera (`pressed`) para notificar al programa principal.
- **Bucle Principal (`loop`):** El programa principal verifica constantemente la bandera `pressed`. Si está activa, imprime un mensaje en el Monitor Serie con el número total de pulsaciones y reinicia la bandera.
- **Desconexión Automática de la Interrupción:** Después de 1 minuto (60000 ms) de ejecución, el programa desactiva la interrupción para demostrar cómo detener la detección de pulsaciones cuando ya no es necesaria.

## 🛠️ Hardware Requerido

- Placa ESP32 (o compatible)
- 1x Botón pulsador (Push Button)
- Cables de conexión

## 🔌 Diagrama de Conexión

Conecta el botón pulsador de la siguiente manera:

- Un terminal del botón al pin **GPIO 18** del ESP32.
- El otro terminal del botón al pin **GND** (tierra) del ESP32.

La resistencia `INPUT_PULLUP` se habilita por software, por lo que no es necesario añadir una resistencia externa.

## 🚀 Cómo Usar

1.  **Clona o descarga** este repositorio en tu computadora.
2.  **Abre el archivo `.ino`** (o el archivo `.cpp` principal) en el Arduino IDE o en PlatformIO.
3.  **Selecciona tu placa ESP32** en el menú de herramientas.
4.  **Conecta tu placa ESP32** al ordenador y sube el código.
5.  **Abre el Monitor Serie** (Herramientas > Monitor Serie) y asegúrate de que la velocidad de baudios esté configurada a **115200**.
6.  **Presiona el botón** varias veces. Verás cómo el contador se incrementa en el monitor serie.
7.  **Espera 1 minuto**. Verás el mensaje "Interrupt Detached!" y las pulsaciones posteriores dejarán de ser contadas.

## 📖 Explicación del Código

```cpp
#include <Arduino.h>

// Estructura para manejar el estado del botón
struct Button {
    const uint8_t PIN;
    uint32_t numberKeyPresses;
    bool pressed;
};

Button button1 = {18, 0, false};

// Función de interrupción (debe ser rápida y estar en IRAM)
void IRAM_ATTR isr() {
    button1.numberKeyPresses += 1;
    button1.pressed = true;
}

void setup() {
    Serial.begin(115200);
    pinMode(button1.PIN, INPUT_PULLUP); // Habilitar pull-up interno
    attachInterrupt(button1.PIN, isr, FALLING); // Adjuntar interrupción
}

void loop() {
    // Si el botón fue presionado, imprimir mensaje
    if (button1.pressed) {
        Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
        button1.pressed = false; // Reiniciar bandera
    }

    // Desconectar interrupción después de 1 minuto (60000 ms)
    static uint32_t lastMillis = 0;
    if (millis() - lastMillis > 60000) {
        lastMillis = millis();
        detachInterrupt(button1.PIN);
        Serial.println("Interrupt Detached!");
    }
}