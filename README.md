# IOT-
#include <dht11.h>

#include <SoftwareSerial.h>
#include <DHT.h>

// Define software serial pins
SoftwareSerial nodemcu(8, 9);

// Define pin numbers
int pinGreenLed = 12;
int pinYellowLed = 11;
int pinSensor = A5;
int DHTPin = A4;  // Pin for DHT11 sensor
int buzzer = 10;

// Threshold for methane sensor
int THRESHOLD = 650;

// DHT sensor settings
#define DHTTYPE DHT11
DHT dht(DHTPin, DHTTYPE);

int rdata = 0;
String mystring;

void setup() {
  Serial.begin(9600);
  nodemcu.begin(9600);

  pinMode(buzzer, OUTPUT);
  pinMode(pinGreenLed, OUTPUT);
  pinMode(pinYellowLed, OUTPUT);
  pinMode(pinSensor, INPUT);

  // Initialize the DHT sensor
  dht.begin();
}

void loop() {
  // Read methane sensor value
  int rdata = analogRead(pinSensor);

  Serial.print("Methane Range: ");
  Serial.println(rdata);

  // Check methane level against threshold
  if(rdata >= THRESHOLD){
    digitalWrite(pinGreenLed, HIGH);
    digitalWrite(pinYellowLed, LOW);
    digitalWrite(buzzer, HIGH);  
    delay(50);
  } else {
    digitalWrite(pinGreenLed, LOW);
    digitalWrite(pinYellowLed, HIGH);
    digitalWrite(buzzer, LOW);
  }
 
  // Read temperature and humidity from DHT11 sensor
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  // Check if readings are valid
  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Failed to read from DHT sensor!");
  } else {
    Serial.print("Humidity: ");
    Serial.print(humidity);
    Serial.print(" %\t");
    Serial.print("Temperature: ");
    Serial.print(temperature);
    Serial.println(" *C");
  }

  // Check for incoming data from NodeMCU
  if (nodemcu.available() > 0) {
    char data = nodemcu.read();
    Serial.println(data);
  }

  // Send data to NodeMCU based on methane level
  if(rdata < THRESHOLD) {
    mystring = "Methane Range: " + String(rdata) + " Humidity: " + String(humidity) + " Temperature: " + String(temperature);
    nodemcu.println(mystring);
    Serial.println(mystring);
  } else {
    mystring = "Food Spoiled";
    nodemcu.println(mystring);
    Serial.println(mystring);
  }

  mystring = "";
  delay(1000);
}
