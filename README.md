#include <TimerOne.h>
#include<Wire.h>
#include<LCD.h>
#include<LiquidCrystal_I2C.h>
#define I2C_ADDR 0x27     // Aдреса на I2C на LCD. Може да се намери с помощта на I2C скенер.
#define BACKLIGHT_PIN 3   //Това са щифтовете между LCD и I2C модула, а не между LCD и Arduino.
#define En_pin  2
#define Rw_pin  1
#define Rs_pin  0
#define D4_pin  4
#define D5_pin  5
#define D6_pin  6
#define D7_pin  7
LiquidCrystal_I2C
 lcd(I2C_ADDR,En_pin,Rw_pin,Rs_pin,D4_pin,D5_pin,D6_pin,D7_pin);
 //Край на раздела за конфигуриране и инициализация на LCD.
int i;
int d = 0;
int d_old = -1;
int sign;
long value;
long pwm;
float result;
//int clockpin = 4;
int clockpin = 9;
//int datapin = 3;
int datapin = 6;
int period = 128;
int pwmpin = A3;
unsigned long tempmicros;

void setup() {
  lcd.begin (16,2);
  lcd.setBacklightPin(BACKLIGHT_PIN,POSITIVE);
  lcd.setBacklight(HIGH);
  lcd.home (); // go home (0, 0)
  lcd.print("Width:");    //Ширина на нишката.
  lcd.setCursor(0, 1);
  lcd.print("Speed:");  //Скорост на ролката. 
  Serial.begin(115200);
  pinMode(clockpin, INPUT);
  pinMode(datapin, INPUT);
  pinMode(pwmpin, INPUT);    
  Timer1.initialize(period);   
  makeOutput(0);
  //Serial.println("Ready");
  Serial.println("Ready: ");
}

void loop () {
  int x = analogRead(pwmpin);
  d = (x/100) * 1;    //Обхватът на оборотите се регулира чрез промяна на множителя.    
  if (d <= 1) d = 1;   //Отървете се от d = 0, за да осигурите достатъчно забавяне, за да може стъпковият двигател да функционира правилно.   
  while (digitalRead(clockpin) == HIGH) {} 
  tempmicros = micros();
  while (digitalRead(clockpin) == LOW) {} 
  if ((micros() - tempmicros) > 500) { 
   decode();
  }

}

void makeOutput(int um) 
{
  double d = (double)um / 100;
  unsigned int value = d / 4.4 * 1023;//4.41v - USB power (Please CHEK IT FOR YOUSELF)
  Timer1.pwm(pwmpin, value); 
}

void decode() {
  sign = 1;
  value = 0;
  for (i = 0; i < 23; i++) {
    while (digitalRead(clockpin) == HIGH) { }
    while (digitalRead(clockpin) == LOW) {} 
    if (digitalRead(datapin) == LOW) {
//    if (analogRead(pwmpin) == LOW) {  

      if (i < 20) {
        value |= 1 << i;

      }

      if (i == 20) {
        sign = -1;

      }

    }

  }

  result = (value * sign) / 2000.00;
  makeOutput(value); // input as micrometer (um)
   interrupts();                      //Временно повторно активиране на прекъсването, за да функционира LCD библиотеката.      
        lcd.setCursor(7, 0);
        lcd.print(result,4);
        lcd.print(" in     ");
        lcd.setCursor(7, 1);
        lcd.print(d);             //Скорост на ролката.           
        lcd.print(" ");
     noInterrupts();              //Деактивирайте временното повторно активирано прекъсване.           
     Serial.println(result, 4);
     delay(1000); 
    result = (value*sign) / 100.00;  
     interrupts();
         lcd.setCursor(7, 0);
         lcd.print(result,2);
         lcd.print(" mm     ");
         lcd.setCursor(7, 1);
         lcd.print(d);                //Скорост на ролката.        
         lcd.print(" ");
      noInterrupts();                //Деактивирайте временното повторно активирано прекъсване.
      Serial.println(result, 2);
      delay(1000); 
}
  
