// C++ code
int points = 0;
int combo = 1;
const int maxCombo = 5;
bool correctButtonPress = false;

void setup()
{
  // Initialize the built-in LED pin as an output
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(9600); // Initialize serial communication for debugging
}

void loop()
{
  // Simulate a correct button press (this should be replaced with actual button press logic)
  correctButtonPress = true; // Change to false or true based on button press logic
  
  if (correctButtonPress) {
    addPoints();
    correctButtonPress = false; // Reset for the next loop iteration
  }
  
  // Simulate delay to mimic real-world scenario
  delay(1000); // Wait for 1000 milliseconds
}

// Function to add points based on the current combo multiplier
void addPoints()
{
  // Calculate the points add
  int pointsToAdd = 10 * combo;
  points += pointsToAdd;
  
  // Increase the combo multiplier, capped at maxCombo of 5
  if (combo < maxCombo) {
    combo++;
  }

  // Print the current points and combo for debugging
  Serial.print("Points: ");
  Serial.print(points);
  Serial.print(", Combo: ");
  Serial.println(combo);
}

// Function to reset combo multiplier
void resetCombo()
{
  combo = 1;
}
