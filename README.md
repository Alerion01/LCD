# LCD
I have a problem with my project, my project is about a counter that must be reproduced on my lcd screen by arduino, but the lcd does not show text, well yes, but sometimes and only are rare symbols, or simple zeros, this is my code:  

#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Inicializa la biblioteca con la dirección I2C y el tamaño de la pantalla (16x2 en este caso)
LiquidCrystal_I2C lcd(0x27, 16, 2);  // La dirección I2C puede variar; comúnmente es 0x27 o 0x3F

int segundos = 0;
int minutos = 0;
int horas = 0;
int totalSegundos = 0; // Contador total de segundos
int totalMinutos = 0; // Contador total de minutos
int totalHoras = 0; // Contador total de horas
int pinSensor = 13; // Cambia esto al pin donde tienes conectado el sensor PIR
unsigned long tiempoAnterior = 0;
unsigned long intervalo = 5000; // Intervalo de inactividad de 5 segundos
bool movimientoDetectado = false;

void setup() {
  Serial.begin(9600);  // Configura la comunicación serial a 9600 baudios
  pinMode(pinSensor, INPUT);
  
  // Inicializa la pantalla LCD
  lcd.init();
  delay(100); // Pequeño tiempo de espera después de la inicialización
  lcd.backlight(); // Activa la luz de fondo del LCD
  
  // Muestra mensaje inicial en la LCD
  lcd.setCursor(0, 0);
  lcd.print("Esperando mov.");
}

void loop() {
  int valorSensor = digitalRead(pinSensor);
  unsigned long tiempoActual = millis();
  
  if (valorSensor == HIGH) {
    if (!movimientoDetectado) {
      // Muestra el mensaje de movimiento detectado solo una vez
      Serial.println("Movimiento detectado");
      lcd.clear(); // Limpia la pantalla LCD
      lcd.setCursor(0, 0);
      lcd.print("Movimiento");
      lcd.setCursor(0, 1);
      lcd.print("detectado");
      movimientoDetectado = true;
      // Reinicia el contador
      segundos = 0;
      minutos = 0;
      horas = 0;
    }
    tiempoAnterior = tiempoActual; // Actualiza el tiempo anterior
  } else if (tiempoActual - tiempoAnterior >= intervalo) {
    if (movimientoDetectado) {
      movimientoDetectado = false; // Reinicia la bandera de movimiento detectado
      // Muestra el mensaje de movimiento no detectado
      Serial.println("Movimiento no detectado");
      lcd.clear(); // Limpia la pantalla LCD
      lcd.setCursor(0, 0);
      lcd.print("No hay mov.");
    }

    // Incrementa el contador
    segundos++;
    totalSegundos++; // Incrementa el contador total de segundos
    if (segundos >= 60) {
      segundos = 0;
      minutos++;
      if (minutos >= 60) {
        minutos = 0;
        horas++;
      }
    }

    if (totalSegundos >= 60) {
      totalSegundos = 0;
      totalMinutos++; // Incrementa el contador total de minutos
      if (totalMinutos >= 60) {
        totalMinutos = 0;
        totalHoras++; // Incrementa el contador total de horas
      }
    }

    // Muestra el contador
    Serial.print("Tiempo: ");
    Serial.print(horas);
    Serial.print(":");
    Serial.print(minutos);
    Serial.print(":");
    Serial.println(segundos);

    // Muestra el contador total
    Serial.print("Tiempo total sin movimiento: ");
    Serial.print(totalHoras);
    Serial.print(":");
    Serial.print(totalMinutos);
    Serial.print(":");
    Serial.println(totalSegundos);

    // Actualiza la pantalla LCD con el contador
    lcd.setCursor(0, 1);
    lcd.print("T: ");
    lcd.print(horas);
    lcd.print(":");
    lcd.print(minutos);
    lcd.print(":");
    lcd.print(segundos);

    delay(1000); // Espera un segundo antes de incrementar el contador
  }
}
