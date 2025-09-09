int const LED_RED = 13;
int const LED_YELLOW = 12;
int const LED_GREEN = 11;

const int BUTTON_PIN = 10;  // Pin del botón

int menuOption = 0;  // Opción actual del menú (1 a 9)
bool lastButtonState = HIGH;       // Último estado leído del botón
bool buttonPressedFlag = false;    // Para evitar múltiples activaciones rápidas
unsigned long lastDebounceTime = 0;
const unsigned long debounceDelay = 50;

// Variables para el fade
int brightness = 0;
int fadeAmount = 5;
unsigned long previousMillis = 0;
const long interval = 10;

void setup() {
  pinMode(LED_RED, OUTPUT);
  pinMode(LED_YELLOW, OUTPUT);
  pinMode(LED_GREEN, OUTPUT);

  pinMode(BUTTON_PIN, INPUT_PULLUP);  // Botón con resistencia interna pull-up

  Serial.begin(9600); // Comunicación con el monitor serial

  Serial.println("=== MENU ===");
  Serial.println("[1] Turn on LED RED");
  Serial.println("[2] Turn off LED RED");
  Serial.println("[3] Turn on LED YELLOW");
  Serial.println("[4] Turn off LED YELLOW");
  Serial.println("[5] Turn on LED GREEN");
  Serial.println("[6] Turn off LED GREEN");
  Serial.println("[7] Turn on ALL");
  Serial.println("[8] Turn off ALL");
  Serial.println("[9] Intermittence (Fade in/out)");
}

void loop() {
  // Lectura y debounce del botón mejorado
  int reading = digitalRead(BUTTON_PIN);

  if (reading != lastButtonState) {
    lastDebounceTime = millis();  // Se detectó cambio, reiniciamos timer debounce
  }

  if ((millis() - lastDebounceTime) > debounceDelay) {
    // Si botón presionado y no se ha procesado esta pulsación aún
    if (reading == LOW && !buttonPressedFlag) {
      menuOption++;
      if (menuOption > 9) menuOption = 1;  // Ciclar del 1 al 9
      ejecutarOpcion(menuOption);
      buttonPressedFlag = true;  // Marcamos pulsación procesada
    }

    // Si botón liberado, reseteamos la bandera para próxima pulsación
    if (reading == HIGH) {
      buttonPressedFlag = false;
    }
  }

  lastButtonState = reading;

  // Leer opción desde Serial
  if (Serial.available() > 0) {
    char optionChar = Serial.read();
    if (optionChar >= '0' && optionChar <= '9') {
      int option = optionChar - '0';
      menuOption = option;  // Actualizamos menuOption para activar el fade si es 9
      ejecutarOpcion(option);
    }
  }

  // Si la opción es 9, hacer el fade continuo igual que en el botón
  if (menuOption == 9) {
    unsigned long currentMillis = millis();
    if (currentMillis - previousMillis >= interval) {
      previousMillis = currentMillis;

      brightness += fadeAmount;

      if (brightness <= 0 || brightness >= 255) {
        fadeAmount = -fadeAmount;
      }

      analogWrite(LED_RED, brightness);
      analogWrite(LED_YELLOW, brightness);
      analogWrite(LED_GREEN, brightness);
    }
  }
}

void ejecutarOpcion(int option) {
  switch (option) {

    case 0: // Toggle LED rojo
      if (digitalRead(LED_RED) == HIGH) {
        digitalWrite(LED_RED, LOW);
        Serial.println("LED RED OFF (Toggle 0)");
      } else {
        digitalWrite(LED_RED, HIGH);
        Serial.println("LED RED ON (Toggle 0)");
      }
      break;

    case 1: // Encender rojo
      digitalWrite(LED_RED, HIGH);
      Serial.println("LED RED ON");
      break;

    case 2: // Apagar rojo
      digitalWrite(LED_RED, LOW);
      Serial.println("LED RED OFF");
      break;

    case 3: // Encender amarillo
      digitalWrite(LED_YELLOW, HIGH);
      Serial.println("LED YELLOW ON");
      break;

    case 4: // Apagar amarillo
      digitalWrite(LED_YELLOW, LOW);
      Serial.println("LED YELLOW OFF");
      break;

    case 5: // Encender verde
      digitalWrite(LED_GREEN, HIGH);
      Serial.println("LED GREEN ON");
      break;

    case 6: // Apagar verde
      digitalWrite(LED_GREEN, LOW);
      Serial.println("LED GREEN OFF");
      break;

    case 7: // Encender todos
      digitalWrite(LED_RED, HIGH);
      digitalWrite(LED_YELLOW, HIGH);
      digitalWrite(LED_GREEN, HIGH);
      Serial.println("ALL LEDs ON");
      break;

    case 8: // Apagar todos
      digitalWrite(LED_RED, LOW);
      digitalWrite(LED_YELLOW, LOW);
      digitalWrite(LED_GREEN, LOW);
      Serial.println("ALL LEDs OFF");
      break;

    case 9: // Intermitencia con fade
      Serial.println("Intermittence (Fade in/out) started");
      break;

    default:
      Serial.println("Invalid option. Choose 0-9");
      break;
  }
}
