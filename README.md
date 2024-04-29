const int motor1Pin1 = 2;  // Motor 1 pins
const int motor1Pin2 = 4;
const int motor2Pin1 = 7;  // Motor 2 pins
const int motor2Pin2 = 8;
const int trigPin = 12;    // Ultrasonic sensor pins
const int echoPin = 13;
const int IR_left = A0;    // IR sensor pins
const int IR_right = A1;
const int maxDistance = 40; // Maximum distance to detect obstacles (in cm)
const int minReverseDistance = 5; // Distance to start reversing (in cm)
const float cmToInch = 0.393701; // Conversion factor from cm to inches
const float reverseDistanceInch = 2.0; // Reverse distance in inches

long duration;
int distance;

void setup() {
  pinMode(motor1Pin1, OUTPUT);
  pinMode(motor1Pin2, OUTPUT);
  pinMode(motor2Pin1, OUTPUT);
  pinMode(motor2Pin2, OUTPUT);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(IR_left, INPUT);
  pinMode(IR_right, INPUT);
  Serial.begin(9600);
}

void loop() {
  int leftSensor = digitalRead(IR_left);
  int rightSensor = digitalRead(IR_right);

  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  duration = pulseIn(echoPin, HIGH);
  distance = duration * 0.034 / 2;

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.print(" cm, Left Sensor: ");
  Serial.print(leftSensor);
  Serial.print(", Right Sensor: ");
  Serial.println(rightSensor);

  if (distance <= maxDistance && distance >= minReverseDistance) {
    if (leftSensor == LOW && rightSensor == LOW) {
      moveForward();
    } else if (leftSensor == LOW) {
      turnLeft();
    } else if (rightSensor == LOW) {
      turnRight();
    } else {
      continueStraight();
    }
  } else if (distance < minReverseDistance && distance >= 0) {
    reverse();
    delay(reverseDuration(distance)); // Delay for the reverse duration
  } else {
    stopMoving();
  }
}

void moveForward() {
  analogWrite(motor1Pin1, 255);
  analogWrite(motor1Pin2, 0);
  analogWrite(motor2Pin1, 255);
  analogWrite(motor2Pin2, 0);
}

void turnRight() {
  analogWrite(motor1Pin1, 255);
  analogWrite(motor1Pin2, 0);
  analogWrite(motor2Pin1, 0);
  analogWrite(motor2Pin2, 255);
}

void turnLeft() {
  analogWrite(motor1Pin1, 0);
  analogWrite(motor1Pin2, 255);
  analogWrite(motor2Pin1, 255);
  analogWrite(motor2Pin2, 0);
}

void stopMoving() {
  analogWrite(motor1Pin1, 0);
  analogWrite(motor1Pin2, 0);
  analogWrite(motor2Pin1, 0);
  analogWrite(motor2Pin2, 0);
}

void continueStraight() {
  analogWrite(motor1Pin1, 255);
  analogWrite(motor1Pin2, 0);
  analogWrite(motor2Pin1, 255);
  analogWrite(motor2Pin2, 0);
}

void reverse() {
  analogWrite(motor1Pin1, 0);
  analogWrite(motor1Pin2, 255);
  analogWrite(motor2Pin1, 0);
  analogWrite(motor2Pin2, 255);
}

float reverseDuration(int distance) {
  return (distance * cmToInch) / reverseDistanceInch * 1000; // Convert distance in cm to inches and calculate the reverse duration in milliseconds
}

