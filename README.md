#include <TM1637Display.h>
#include <IRremote.h>

// Pins for TM1637 4-digit 7-segment display
#define CLK 11
#define DIO 12

// Pins for 3x3 LED matrix
const int ledMatrixRows[] = {6, 7, 5};
const int ledMatrixCols[] = {4, 3, 2};

// Pin for IR receiver
#define RECV_PIN 10

// Pin for buzzer
#define BUZZER_PIN 8

// Initialize the TM1637 display
TM1637Display display(CLK, DIO);

// Initialize the IR receiver
IRrecv irrecv(RECV_PIN);
decode_results results;

// Game variables
int score = 0;
int comboMultiplier = 1;
int lives = 5;
unsigned long ledOnTime = 3000; // LED on time starts at 3 seconds
unsigned long previousMillis = 0;
int currentLEDRow = -1;
int currentLEDCol = -1;
bool gameActive = false;
bool ledActive = false;
const unsigned long initialLedOnTime = 3000;
const unsigned long minLedOnTime = 500;

// Hex codes for IR remote buttons
#define START 0xFD00FF
#define BUTTON_1 0xFD08F7
#define BUTTON_2 0xFD8807
#define BUTTON_3 0xFD48B7
#define BUTTON_4 0xFD28D7
#define BUTTON_5 0xFDA857
#define BUTTON_6 0xFD6897
#define BUTTON_7 0xFD18E7
#define BUTTON_8 0xFD9867
#define BUTTON_9 0xFD58A7

void setup() {
  Serial.begin(9600);
  // Initialize LED matrix pins
  for (int i = 0; i < 3; i++) {
    pinMode(ledMatrixRows[i], OUTPUT);
    pinMode(ledMatrixCols[i], OUTPUT);
  }
  // Initialize buzzer pin
  pinMode(BUZZER_PIN, OUTPUT);

  // Initialize the display
  display.setBrightness(7);
  display.showNumberDec(0);

  // Initialize the IR receiver
  irrecv.enableIRIn();

  // Initialize random seed
  randomSeed(analogRead(0));
}

void loop() {
  if (!gameActive) {
    standbyMode();
    if (irrecv.decode(&results)) {
      if (results.value == START) {
        startGame();
      }
      irrecv.resume(); // Receive the next value
    }
  } else {
    if (ledActive && (millis() - previousMillis >= ledOnTime)) {
      loseLife();
      nextLED();
    }
    if (irrecv.decode(&results)) {
      handleInput(results.value);
      irrecv.resume(); // Receive the next value
    }
  }
}

void standbyMode() {
  static unsigned long standbyMillis = 0;
  static int currentStandbyLED = 0;
  if (millis() - standbyMillis >= 500) {
    standbyMillis = millis();
    clearLEDs();
    digitalWrite(ledMatrixCols[currentStandbyLED], HIGH);
    for (int row = 0; row < 3; row++) {
      digitalWrite(ledMatrixRows[row], LOW);
    }
    currentStandbyLED = (currentStandbyLED + 1) % 3;
  }
}

void startGame() {
  gameActive = true;
  score = 0;
  comboMultiplier = 1;
  lives = 5;
  ledOnTime = initialLedOnTime;
  display.showNumberDec(0);
  nextLED();
}

void nextLED() {
  clearLEDs();
  currentLEDRow = random(0, 3);
  currentLEDCol = random(0, 3);
  digitalWrite(ledMatrixCols[currentLEDCol], HIGH);
  digitalWrite(ledMatrixRows[currentLEDRow], LOW);
  previousMillis = millis();
  ledActive = true;
}

void clearLEDs() {
  for (int row = 0; row < 3; row++) {
    digitalWrite(ledMatrixRows[row], HIGH);
  }
  for (int col = 0; col < 3; col++) {
    digitalWrite(ledMatrixCols[col], LOW);
  }
}

void handleInput(unsigned long input) {
  int button = -1;
  switch (input) {
    case BUTTON_1: button = 1; break;
    case BUTTON_2: button = 2; break;
    case BUTTON_3: button = 3; break;
    case BUTTON_4: button = 4; break;
    case BUTTON_5: button = 5; break;
    case BUTTON_6: button = 6; break;
    case BUTTON_7: button = 7; break;
    case BUTTON_8: button = 8; break;
    case BUTTON_9: button = 9; break;
  }
  if (button != -1) {
    int pressedRow = (button - 1) / 3;
    int pressedCol = (button - 1) % 3;
    if (pressedRow == currentLEDRow && pressedCol == currentLEDCol) {
      score += 10 * comboMultiplier;
      Serial.print("Score: ");
      Serial.println(score);
      display.clear();
      display.showNumberDec(score);
      if (millis() - previousMillis <= 1000) { // Check if within the first second
        comboMultiplier++;
      }
      // tone(BUZZER_PIN, 1000, 200); // High pitch chime
      if (score % 100 == 0 && ledOnTime > minLedOnTime) {
        ledOnTime -= 500;
      }
      nextLED();
    } else {
      loseLife();
      nextLED();
    }
  }
}

void loseLife() {
  lives--;
  comboMultiplier = 1;
  // tone (BUZZER_PIN, 500, 200); // Low pitch chime
  if (lives <= 0) {
    gameOver();
  }
}

void gameOver() {
  gameActive = false;
  clearLEDs();
  display.showNumberDec(score);
}
