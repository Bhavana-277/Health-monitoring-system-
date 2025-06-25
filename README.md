# Health-monitoring-system-
#include <Adafruit_GFX.h>   // include adafruit GFX library

#include <KS0108_GLCD.h>    // include KS0108 GLCD library



void(* resetFunc) (void) = 0;

KS0108_GLCD display = KS0108_GLCD(53, 51, 49, 47, 45, 43, 41, 39, 37, 35, 33, 31, 29, 27);



#include <Adafruit_MLX90614.h>

Adafruit_MLX90614 mlx = Adafruit_MLX90614();

#include "HX711.h"



// HX711 circuit wiring

const int LOADCELL_DOUT_PIN = 4;

const int LOADCELL_SCK_PIN = 5;



HX711 scale;

float calibration_factor = 21000; // Adjust this calibration factor initially



#define trig 8

#define echo 9



#define buzz 13



#define up 52

#define down 48

#define ok 50

float duration = 0;

float distance = 0;



void setup()

{

  pinMode(buzz, OUTPUT);

  Serial.begin(9600);

  Serial1.begin(9600);

  scale.begin(LOADCELL_DOUT_PIN, LOADCELL_SCK_PIN);

  mlx.begin();



  digitalWrite(buzz, HIGH);



  // initialize KS0108 GLCD module with active high CS pins

  if ( display.begin(KS0108_CS_ACTIVE_HIGH) == false )

  {

    Serial.println( F("display initialization failed!") );    // lack of RAM space

    while (true); // stay here forever!

  }

  display.clearDisplay();

  display.setTextSize(2); //Draw 2X-scale text

  display.setTextColor(KS0108_ON);

  display.setCursor(23, 5);

  display.println("WELCOME"); 

  display.setCursor(53, 24);

  display.println("TO");

  display.setCursor(30, 44);

  display.println("VIGNAN");

  display.display();

  delay(2000); // Pause for 2 seconds

  digitalWrite(buzz, LOW);

display.clearDisplay();

  display.setTextSize(2); //Draw 2X-scale text

  display.setTextColor(KS0108_ON); //KS0108_OFF, KS0108_ON

  display.setCursor(10, 5);

  display.println("Deparment"); 

  display.setCursor(53, 24);

  display.println("OF");

  display.setCursor(45, 44);

  display.println("ECE");

  display.display();

  delay(2000);



  

  pinMode(trig, OUTPUT);

  pinMode(echo, INPUT);



  pinMode(up, INPUT_PULLUP);

  pinMode(down, INPUT_PULLUP);

  pinMode(ok, INPUT_PULLUP);



  //  Serial.println("HX711 calibration sketch");

  //  Serial.println("Remove all weight from scale");

  //  Serial.println("After readings begin, place known weight on scale");

  //  Serial.println("Press + or a to increase calibration factor");

  //  Serial.println("Press - or z to decrease calibration factor");



  scale.set_scale();

  scale.tare();  // Reset the scale to 0



  long zero_factor = scale.read_average();  // Get a baseline reading

  Serial.print("Zero factor: ");

  Serial.println(zero_factor);



}





float weight = 0;

float height = 0;

float BMI = 0;

float height_ft = 0;

float body_fat = 0;

volatile boolean newData = false;

int HBM;

int Spo2;

int st = 0;

int Age = 0;

int gender = 0;

void loop()

{

Reset:

  st = 0;

  gender = 0;

  Age = 0;

  while (1)

  {

    while (st < 3)

    {

      if (digitalRead(ok) == LOW)

      {

        buz(10);

        while (digitalRead(ok) == LOW);

        st++;

      }

      if (st == 0)

      {

        display.clearDisplay();

        display.setTextSize(2); //Draw 2X-scale text

        display.setTextColor(KS0108_ON);

        display.setCursor(5, 5);

        display.println("Set Gender");

        if (digitalRead(up) == LOW)

        {

          buz(10);

          gender = 0;

        }

        if (digitalRead(down) == LOW)

        {

          buz(10);

          gender = 1;

        }

        if (gender == 0)

        {

          display.setTextSize(3);

          display.setCursor(10, 30);

          display.println("Female");

        }

        if (gender == 1)

        {

          display.setTextSize(3);

          display.setCursor(25, 30);

          display.println("Male  ");

        }

        display.display();

        delay(50);

      }

      if (st == 1)

      {

        display.clearDisplay();

        display.setTextSize(2); //Draw 2X-scale text

        display.setTextColor(KS0108_ON);

        display.setCursor(5, 5);

        display.println("Set Ur Age");

        if (digitalRead(up) == LOW)

        {

          buz(10);

          Age++;

        }

        if (digitalRead(down) == LOW)

        {

          buz(10);

          Age--;

        }



        if (Age < 0)

          Age = 0;

        int digitCount = String(Age).length();

        display.setTextSize(3);

        display.setCursor((128 - (10 * digitCount)) / 2, 30);

        display.println(Age);

        display.display();

        delay(50);

      }

      if (st == 2)

      {

        display.clearDisplay();

        display.setTextSize(2);

        display.setCursor(14, 30);

        display.println("Click OK");

        display.display();

        delay(100);

      }



    }

    if (Serial1.available() > 0)

    {

      // If there is data available to read

      String data = Serial1.readStringUntil('\n');

      int commaIndex = data.indexOf(',');



      // Extract sensor data

      if (commaIndex != -1)

      {

        HBM = data.substring(0, commaIndex).toInt();

        Spo2 = data.substring(commaIndex + 1).toInt();



        // Indicate new data received

        newData = true;

      }

    }



    scale.set_scale(calibration_factor);  // Adjust to this calibration factor

    Serial.print("Reading: ");

    weight = scale.get_units();

    if (weight < 0)

      weight = 0;

    get_distance();

    get_BMI();



    Serial.print("Weight:");

    Serial.print(weight, 1);

    Serial.print("kg");



    Serial.print("  Dis:");

    Serial.print(distance);

    Serial.print("cm");



    Serial.print("  Height:");

    Serial.print(height);

    Serial.print("m");



    Serial.print("  Height:");

    Serial.print(height_ft);

    Serial.print("ft");



    Serial.print("  BMI:");

    Serial.print(BMI);

    Serial.print("kg/m2");



    Serial.print("  Body Temp:");

    Serial.print(mlx.readObjectTempF());

    Serial.println();



    if (newData)

    {

      Serial.print("BPM:");

      Serial.print(HBM);

      Serial.print("  Spo2:");

      Serial.println(Spo2);

      newData = false;

    }

    else

    {

      HBM = 0;

      Spo2 = 0;

    }



    display_info();

    delay(500);

    if (digitalRead(ok) == LOW)

    {

      buz(500);

      while (digitalRead(ok) == LOW);

      goto Reset;

    }



  }

}



void display_info()

{

  display.clearDisplay();

  display.setTextSize(2); // Draw 2X-scale text

  display.setTextColor(KS0108_ON);

  display.setCursor(0, 0);

  display.println(("Wt:" + String(weight, 1)) + "KG");

  display.setCursor(0, 16);

  display.println(("Ht:" + String(height_ft, 1)) + "ft");

  display.setCursor(0, 32);

  display.println(("BMI:" + String(BMI, 1)));

  //display.setTextSize(1); display.print("kg/m2");

  display.setTextSize(2);

  display.setCursor(0, 48);

  display.println(("Fat:" + String(body_fat, 1)) + "%");

  display.display();      // Show initial text

  delay(2000);



  display.clearDisplay();

  display.setTextSize(2); // Draw 2X-scale text

  display.setTextColor(KS0108_ON);

  display.setCursor(0, 0);

  display.println(("SPO2:" + String(Spo2)) + "%");

  display.setCursor(0, 16);

  display.println(("HBM:" + String(HBM)));

  display.setCursor(0, 32);

  display.println(("Temp:" + String(mlx.readObjectTempF(), 1)) + "F");

  display.display();

  delay(2000);

}



void get_distance()

{

  digitalWrite(trig, LOW); delayMicroseconds(2);

  digitalWrite(trig, HIGH); delayMicroseconds(10); digitalWrite(trig, LOW);

  duration = pulseIn(echo, HIGH);

  distance = duration * 0.0343 / 2;  //distance in cm



  float Person_height = 195 - distance;

  height = Person_height * 0.01;  //meters convertion 1cm = 0.01m

  height_ft = Person_height / 2.54; //inches convertion 1inch = 2.54cm

  height_ft = height_ft / 12;     //feet convertion 1ft = 12inch

}



void get_BMI()

{

  BMI = weight / (height * height);

  if (BMI < 0)

    BMI = 0;

  Serial.println("BMI:" + String(BMI) + "kg/m2");



  body_fat = 1.20 * BMI + 0.23 * Age - 10.8 * gender - 5.4;

  if (body_fat <= 0)

  {

    body_fat = 0;

  }

}



void buz(int val)

{

  digitalWrite(buzz, HIGH);

  delay(val);

  digitalWrite(buzz, LOW);

  delay(val);

}

