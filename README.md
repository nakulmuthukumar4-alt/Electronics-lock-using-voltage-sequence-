

const int sensorPin = A0; 
const int lockPin = 8;

float sequence[] = {1.0, 3.0, 2.0};   // voltage sequence
int seqLength = 3;

float tolerance = 0.2;   // ±0.2V allowed
int currentStep = 0;

unsigned long lastTime = 0;
int holdTime = 800;      // Time (ms) to confirm a voltage

void setup() {
  Serial.begin(9600);
  pinMode(lockPin, OUTPUT);
  digitalWrite(lockPin, LOW);  // lock closed
}

void loop() {

  int adcValue = analogRead(sensorPin);
  float voltage = adcValue * (5.0 / 1023.0); // map ADC → voltage

  Serial.print("Voltage: ");
  Serial.println(voltage);

  // Check if voltage matches the required step
  if (voltage >= (sequence[currentStep] - tolerance) &&
      voltage <= (sequence[currentStep] + tolerance)) {

      if (millis() - lastTime > holdTime) {
        Serial.print("Step ");
        Serial.print(currentStep + 1);
        Serial.println(" OK");

        currentStep++;
        lastTime = millis();
      }

      delay(100);
  } 

  // When full sequence is matched
  if (currentStep == seqLength) {
    Serial.println("Unlocked!");
    digitalWrite(lockPin, HIGH);   // Open lock
    delay(3000);
    digitalWrite(lockPin, LOW);    // Lock again
    currentStep = 0;               // Reset sequence
  }
}
