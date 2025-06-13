#define DAC_PIN 25        // Output pin (DAC1)
#define BUTTON_PIN 4      // Push button pin

enum Waveform { SINE, SQUARE, TRIANGLE };
Waveform currentWave = SINE;

const int tableSize = 100;          // Number of samples
uint8_t sineTable[100];             // Sine lookup table
float frequency = 1000.0;           // Hz
unsigned long lastButtonTime = 0;   // For debounce
int waveIndex = 0;

void generateSineTable() {
  for (int i = 0; i < tableSize; i++) {
    sineTable[i] = (uint8_t)(127.5 + 127.5 * sin(2 * PI * i / tableSize));
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  generateSineTable();
  Serial.println("Function Generator Started");
}

void loop() {
  static int i = 0;

  // Check for button press to change waveform
  if (digitalRead(BUTTON_PIN) == LOW && millis() - lastButtonTime > 300) {
    waveIndex = (waveIndex + 1) % 3;
    currentWave = (Waveform)waveIndex;
    lastButtonTime = millis();

    switch (currentWave) {
      case SINE: Serial.println("Waveform: SINE"); break;
      case SQUARE: Serial.println("Waveform: SQUARE"); break;
      case TRIANGLE: Serial.println("Waveform: TRIANGLE"); break;
    }
  }

  // Generate wave
  switch (currentWave) {
    case SINE:
      dacWrite(DAC_PIN, sineTable[i]);
      break;

    case SQUARE:
      dacWrite(DAC_PIN, (i < tableSize / 2) ? 255 : 0);
      break;

    case TRIANGLE:
      dacWrite(DAC_PIN, (i < tableSize / 2) ?
        (i * 255 / (tableSize / 2)) :
        (255 - ((i - tableSize / 2) * 255 / (tableSize / 2))));
      break;
  }

  i = (i + 1) % tableSize;

  // Adjust timing for frequency
  delayMicroseconds(1000000 / (frequency * tableSize));
}
