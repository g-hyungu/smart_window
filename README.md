# smart_window
/*Arduino project. total code*/




#include <DHT.h>
#define DHTPIN 8 
#define DHTTYPE DHT22
const int sensorPin = 13;   // Analog input pin for the dust sensor
const int digitalPin = 2;   // Digital pin connected to the raindrop module's digital output
const int analogPin = A0;   // Analog pin connected to the raindrop module's analog output
// Motor A 연결
const int enA = 10;
const int in1 = 9;
const int in2 = 6;
// Motor B 연결
const int enB = 3;
const int in3 = 5;
const int in4 = 4;
bool bOpen = true;                              // 창문 열린상태, 블라인드 올라간 상태


DHT dht(DHTPIN, DHTTYPE);



void setup() {  
    Serial.begin(9600);
    Serial.println("DHT22 Sensor Test");
    Serial.println("GP2Y1023AU0F Dust Sensor Test");
    Serial.println("LM393 Raindrop Sensor Module Test");

    dht.begin();
    pinMode(digitalPin, INPUT);
    pinMode(2, OUTPUT);

    // Set motor control pins as outputs
    pinMode(enA, OUTPUT);
    pinMode(in1, OUTPUT);
    pinMode(in2, OUTPUT);
    pinMode(enB, OUTPUT);
    pinMode(in3, OUTPUT);
    pinMode(in4, OUTPUT);
}



void loop() {
    delay(1000);  // Delay between readings

    float temperature = dht.readTemperature();  // Read temperature in Celsius
    float humidity = dht.readHumidity();        // Read humidity
    int sensorValue = analogRead(sensorPin); // Read analog input
    float voltage = sensorValue * (5.0 / 1024.0); // Convert sensor value to voltage
    float dustDensity = 0.17 * voltage - 0.1;      // Conversion formula (may vary, refer to sensor datasheet)
    int rainIntensity = analogRead(analogPin);

    Serial.print("Temperature: ");
    Serial.print(temperature);
    Serial.print("°C\t");
    Serial.print("Humidity: ");
    Serial.print(humidity);
    Serial.println("%");
    Serial.print("Dust concentration: ");
    Serial.print(dustDensity);
    Serial.println(" mg/m³");
    Serial.print("bOpen: ");
    Serial.println(bOpen);

    delay(1000);  // Delay between readings

   // 온도변화에 따른 창문, 블라인드 움직이기
    if (temperature > 30 && bOpen == true) { // 온도가 30도 이상이면 창문 닫고 블라인드 내리기
        motorControl(enA, in1, in2, -50);
        delay(250);
        motorControl(enA, in1, in2, 0);
        motorControl(enB, in3, in4, -50);
        delay(250);
        motorControl(enB, in3, in4, 0);

        bOpen = false;
    }
    else if (humidity < 45 && temperature < 20 && bOpen == false) {  // 창문 열고 블라인드 올리기
        motorControl(enB, in3, in4, 50);
        delay(700);
        motorControl(enB, in3, in4, 0);
        motorControl(enA, in1, in2, 50);
        delay(250);
        motorControl(enA, in1, in2, 0);

        bOpen = true;
    }
    
 
    // 습도 
    if (humidity > 70 && bOpen == true) {// 습도가 45% 이상이면 창문 닫기
        motorControl(enA, in1, in2, -50);  // 창문 닫기
        delay(250);
        motorControl(enA, in1, in2, 0);
        motorControl(enB, in3, in4, 50); // 블라인드 내리기
        delay(250);
        motorControl(enB, in3, in4, 0);

        bOpen = false;
    }
   /* else if(humidity < 45 && bOpen == false) {// 습도가 30보다 작으면 창문 열기
        motorControl(enB, in3, in4, 50);
        delay(700);
        motorControl(enB, in3, in4, 0);
        motorControl(enA, in1, in2, 50);
        delay(450); // 창문 열기
        motorControl(enA, in1, in2, 0);

        bOpen = true;
    }*/
    

    // 미세먼지 감지되면 창문 열기
    if (humidity < 65 && dustDensity > 0.15 && bOpen == false) {
        motorControl(enB, in3, in4, 50);
        delay(700);
        motorControl(enB, in3, in4, 0);
        motorControl(enA, in1, in2, 50);
        delay(500); // 창문 열기
        motorControl(enA, in1, in2, 0);

        bOpen = true;
    }
/*
    // 빗물이 감지되면 창문 닫기 (센서가 잘 작동하면 남겨놓고, 잘 작동하지 않으면 폐기)
    if (rainIntensity > 10 && bOpen == true) {
        motorControl(enA, in1, in2, 50);
        delay(300);
        motorControl(enA, in1, in2, 0);
        motorControl(enB, in3, in4, 50);
        delay(150);
        motorControl(enB, in3, in4, 0);

        bOpen = false;
    }
    else if (rainIntensity == 0 && bOpen == false) {
        motorControl(enB, in3, in4, 50);
        delay(150);
        motorControl(enB, in3, in4, 0);
        motorControl(enA, in1, in2, 50);
        delay(300);
        motorControl(enA, in1, in2, 0);

        bOpen = true;
    }*/
}


// 모터 작동을 위한 함수 (모터  A는 창문, 모터 B는 블라인드 제어)
void motorControl(int enablePin, int in1Pin, int in2Pin, int speed ) {
    analogWrite(enablePin, speed);  // Set motor speed

    if (speed > 0) {
        digitalWrite(in1Pin, HIGH);  // Motor forward
        digitalWrite(in2Pin, LOW);
    } 
    else {
        digitalWrite(in1Pin, LOW);   // Motor backward
        digitalWrite(in2Pin, HIGH);
    }
}
