# ADDAT
#include <SPI.h>
#include <Wire.h>
#include <SoftwareSerial.h> // This library allows for seial communication on other digital
                            // pins of the Arduino (rather than just using pins 0 and 1)
                            // note: Arduino UART has a 64 byte serial buffer!
                           
#include <EEPROM.h>
#include <Servo.h>
#include <LiquidCrystal.h>   // This library allows for use of LCD.                            
#include <Wire.h>
 int TIME_UNTIL_WARMUP = 10;
unsigned long time;
 int analogPin = 0;
 int val = 0;
 int pos;
 int redLED = 6;
 int yellowLED = 7;
 int greenLED = 8;
int reset = 12, enable = 11, lcdD4 = 5, lcdD5 = 4, lcdD6 = 3, lcdD7 = 2;

Servo myservo1;
Servo myservo2;
LiquidCrystal LCDboard(reset, enable, lcdD4, lcdD5, lcdD6, lcdD7);

void setup()   {                
Serial.begin(115200);//sets the baud rate
 myservo1.attach(3);
 myservo2.attach(4);
 pinMode(redLED,OUTPUT);
 pinMode(yellowLED,OUTPUT);
 pinMode(greenLED,OUTPUT);
}
void loop() {  
 
  delay(100);
  val = readAlcohol();
  printTitle();
  printWarming();
  time = millis()/1000;
 
  if(time<=TIME_UNTIL_WARMUP)
  {
    time = map(time, 0, TIME_UNTIL_WARMUP, 0, 100);
    Serial.print(time);
  }else
  {
     printTitle();
     printAlcohol(val);
     printAlcoholLevel(val);  
  }
}
void printTitle()
{
  Serial.println("Breath Analyzer");
}
void printWarming()
{
  Serial.println("Warming up");
  digitalWrite(yellowLED,HIGH);
  delay(5000);
}
void printAlcohol(int value)
{
  Serial.println(val);
}
void printAlcoholLevel(int value)
{

 for(pos = 0; pos >= 0; pos -= 1)  // stays at 0 degrees
  myservo1.write(pos);// tell servo to go to position in variable 'pos'
  myservo2.write(pos);
   //delay(1000);// waits 1sec for the servo to reach the position

  if(value<25)
  {
     LCDboard.blink();
     delay(1000);
     LCDboard.noBlink();  
     LCDboard.print("You are sober.");
     delay(1000);
     LCDboard.setCursor(0,1); // move cursor to the next line
     LCDboard.clear();
     for(pos = 0; pos <= 120; pos += 1)  // stays at 0 degrees
     myservo1.write(pos);// tell servo to go to position in variable 'pos'
     myservo2.write(pos);
     delay(20000);// waits 10sec for the servo to reach the position
     digitalWrite(greenLED,HIGH);
     delay(5000);
  }
  if (value>=100 && value<150)
  {
      Serial.print("You had a beer.");
     
  }
  if (value>=151 && value<200)
  {
      Serial.print("Two or more beers.");
  }
  if (value>=181 && value <450)
  {
      Serial.print("You are not fit to drive!");
     
  }
  if(value>= 450)
  {
     LCDboard.blink();
     delay(1000);
     LCDboard.noBlink();  
     LCDboard.print("You are drunk!");
     delay(1000);
     LCDboard.setCursor(0,2); // move cursor to the next line
     LCDboard.clear();
     digitalWrite(redLED,HIGH);
     delay(5000);
  }
 }
 
 int readAlcohol()
 {
  int val = 0;
  int val1;
  int val2;
  int val3;
  val1 = analogRead(analogPin);
  delay(10);
  val2 = analogRead(analogPin);
  delay(10);
  val3 = analogRead(analogPin);
 
  val = (val1+val2+val3)/3;
  return val;
 }
