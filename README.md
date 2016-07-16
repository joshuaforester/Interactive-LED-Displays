# Interactive-LED-Displays
//Standard Arduino Libraries*****************
#include <avr/interrupt.h>
//*******************************************

//NeoPixel Libraries & Definitions***********
#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
#include <avr/power.h>
#endif
#define PIN 6
//*******************************************

//MPR121 Libraries***************************
#include <Wire.h>
#include <Adafruit_MPR121.h>
//*******************************************

//Camera Libraries***************************
#include <Adafruit_VC0706.h>
#include <SPI.h>
#include <SoftwareSerial.h>
//*******************************************

//Object Creation****************************
Adafruit_NeoPixel strip = Adafruit_NeoPixel(684, PIN, NEO_GRB + NEO_KHZ800);
Adafruit_MPR121 buttonSense = Adafruit_MPR121();
#if ARDUINO >= 100
SoftwareSerial cameraconnection = SoftwareSerial(69, 3);
#else
NewSoftSerial cameraconnection = NewSoftSerial(69, 3);
#endif
Adafruit_VC0706 cam = Adafruit_VC0706(&cameraconnection);
//*******************************************

//Global Variables***************************
volatile bool capChange = false;
volatile uint16_t lasttouched = 0;
volatile uint16_t currtouched = 0;
//*******************************************

//Color Array********************************
volatile uint32_t colors[12] = {strip.Color(0, 0, 255), //Blue
                                strip.Color(255, 0, 0), //Red
                                strip.Color(0, 255, 0), //Green
                                strip.Color(160, 32, 240), //Purple
                                strip.Color(255, 20, 147), //Pink
                                strip.Color(255, 140, 0), //Orange
                                strip.Color(176, 48,96), //Maroon
                                strip.Color(255, 255, 0), //Yellow
                                strip.Color(0, 100, 0), //Dark Green
                                strip.Color(0, 255, 255), //Cyan
                                strip.Color(255, 255, 255), //White
                                strip.Color(25, 25, 112), //Midnight Blue
                                //strip.Color(139, 69, 19) //Brown
                                };
//*******************************************

void setup() {
  while (!Serial);
  //Interrupt Setup**************************
  pinMode(2, INPUT);
  digitalWrite(2, HIGH);
  attachInterrupt(digitalPinToInterrupt(2), pin2ISR, FALLING);
  //*****************************************
  
  Serial.begin(9600);
  
  //Test for Touch Sensor Connection*********
  Serial.println("Adafruit MPR121 Capacitive Touch Sensor Test");
  if(!buttonSense.begin(0x5A))
  {
    Serial.println("MPR121 not found!");
    while(1);
  }
  Serial.println("MPR121 found!");
  //*****************************************

  //Test for Camera Connecction**************
  if (cam.begin()) {
    Serial.println("Camera Found:");
  } else {
    Serial.println("No camera found?");
    return;
  }
  cam.setImageSize(VC0706_320x240);
  cam.setMotionDetect(true);
  Serial.print("Motion detection is ");
  if (cam.getMotionDetect()) 
    Serial.println("ON");
  else 
    Serial.println("OFF");
  //*****************************************

  //NeoPixel Initialization******************
  strip.begin();
  strip.show();
  //*****************************************
  randomSeed(analogRead(A1));
}

void loop() {
  if (capChange)
  {
    currtouched = buttonSense.touched();
    capChange = false;
  }
  if(currtouched == 0 && lasttouched != 0)
  {
    if(lasttouched == _BV(0))
    {
      crazyColors(10);
    }
    if(lasttouched == _BV(1))
    {
      rainbow(5);
    }
    if(lasttouched == _BV(2))
    {
      cameraFunction();
    }
    if(lasttouched == _BV(3))
    {
      theaterChaseRainbow(15);
    }
  }
  lasttouched = currtouched;
}

void pin2ISR(){
  capChange = true;
}

void crazyColors(uint8_t wait){
  for(int i = 0; (i < strip.numPixels() && !capChange); i++){
     int pixelNum1 = random(0,684);
     int pixelNum2 = random(0,684);
     int pixelNum3 = random(0,684);
     strip.setPixelColor(pixelNum1, colors[random(0,13)]);
     strip.setPixelColor(pixelNum2, colors[random(0,13)]);
     strip.setPixelColor(pixelNum3, colors[random(0,13)]);
     strip.show();
     delay(wait);
     strip.setPixelColor(pixelNum1, 0, 0, 0);
     strip.setPixelColor(pixelNum2, 0, 0, 0);
     strip.setPixelColor(pixelNum3, 0, 0, 0);
     strip.show();
   }
}

void rainbow(uint8_t wait) {
  uint16_t i, j;
  for(int k = 0; k < 5; k++)
  {
    for(j=0; (j<256 && !capChange); j++) {
      for(i=0; (i<strip.numPixels() && !capChange); i++) {
        strip.setPixelColor(i, Wheel(((i%38)+j) & 255));
      }
       strip.show();
      delay(wait);
    }
  }
  for(i = 0; i < strip.numPixels(); i++)
  {
    strip.setPixelColor(i, 0, 0, 0);
  }
  strip.show();
}

void theaterChase(uint32_t c, uint8_t wait) {
  int k = 0;
  for (int j=0; (j<10 && !capChange); j++) {  //do 10 cycles of chasing
    for (int q=0; (q < 3 && !capChange); q++) {
      for (uint16_t i=0; (i < strip.numPixels() && !capChange); i=i+3) {
        strip.setPixelColor(i+q, c);    //turn every third pixel on
      }
      strip.show();

      delay(wait);

      for (uint16_t i=0; i < strip.numPixels(); i=i+3) {
        strip.setPixelColor(i+q, 0);        //turn every third pixel off
      }
    }
  }
}

void theaterChaseRainbow(uint8_t wait) {
  for (int j=0; (j < 256 && !capChange); j++) {     // cycle all 256 colors in the wheel
    for (int q=0; (q < 3 && !capChange); q++) {
      for (uint16_t i=0; (i < strip.numPixels() && !capChange); i=i+3) {
        strip.setPixelColor(i+q, Wheel( (i+j) % 255));    //turn every third pixel on
      }
      strip.show();

      delay(wait);

      for (uint16_t i=0; i < strip.numPixels(); i=i+3) {
        strip.setPixelColor(i+q, 0);        //turn every third pixel off
      }
    }
  }
  for(int i = 0; i < strip.numPixels(); i++)
  {
    strip.setPixelColor(i, 0, 0, 0);
  }
  strip.show();
}

void cameraFunction()
{
  int i = 0;
  while(!capChange)
  {
    cam.setMotionDetect(true);
    if (cam.motionDetected())
    {
      cam.setMotionDetect(false);
      i++;
    }
    theaterChase(colors[i], 25);
    if(i >= 10)
    {
      i = 0;
    }
  }
}

uint32_t Wheel(byte WheelPos) {
  WheelPos = 255 - WheelPos;
  if(WheelPos < 85) {
    return strip.Color(255 - WheelPos * 3, 0, WheelPos * 3);
  }
  if(WheelPos < 170) {
    WheelPos -= 85;
    return strip.Color(0, WheelPos * 3, 255 - WheelPos * 3);
  }
  WheelPos -= 170;
  return strip.Color(WheelPos * 3, 255 - WheelPos * 3, 0);
}
