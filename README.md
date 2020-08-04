# C1803
Room Security system
#include <Wire.h> //import Wire library
#include <LiquidCrystal.h>//import liquid crstal library
#include <Keypad.h>//import keypad library
#include <Servo.h> //import Servo library
#define Password_Length 8 //defining the password length

Servo myservo;
char Data[Password_Length]; 
char CPass[Password_Length] = "123A456";//initializing the password
char MPass[Password_Length] = "7655841";//initializing the master password
byte data_count = 0, master_count = 0;
bool Pass_is_good;
char customKey;
int attempt=1;//initializing the attempts

//const - constant
//variable qualifier, modifies the behaviour of variable, read-only
//byte: stores an 8 bit unsigned number
const byte ROWS = 4;
const byte COLS = 4;

//arrangement of keys in the 4x4 alphanumeric keypad
char hexaKeys[ROWS][COLS] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};

byte rowPins[ROWS] = {13,12,11,10};//the digital pins to which row pins of keypad are coonected 
byte colPins[COLS ] = {9,8,7,6};//the digital pins to which row pins of keypad are coonected 

Keypad customKeypad = Keypad(makeKeymap(hexaKeys), rowPins, colPins, ROWS, COLS);  //keymapping, pin_row, pin_col,nos_rows,nos_cols

//CREATE INSTANCE,PASS PIN NUMBERS
//RS,EN,D4,D5,D6,D7
LiquidCrystal lcd(14,15,16,17,18,19); 

void setup(){
  lcd.begin(16,2);//initializing the lcd which has col:16,row:2
  lcd.setCursor(5,0); //col:5;row:0
  lcd.print("Hello");
  pinMode(5, OUTPUT);//digital pin to which green led is connected and here it's an o/p pin 
  pinMode(4, OUTPUT);//digitalpin to which red led is connected and here it's an o/p pin 
  pinMode(3,OUTPUT);//digitalpin to which buzzer is connected and here it's an o/p pin 
  myservo.attach(2);//digital pin to which servo motor is connected
  ServoClose();//calling the function void ServoClose()
  delay(1000);//wait for 1s
}

void loop(){
 
  lcd.setCursor(0,0); //col:0;row:0
  lcd.print("Enter Password:");

  
  customKey = customKeypad.getKey();
  if (customKey)  // if some key is received or pressed
  {
    Data[data_count] = customKey; //the key entered or received is stored in data
    lcd.setCursor((5+data_count),1); 
    lcd.print(Data[data_count]); //printing the entered key
    data_count++;//incrementing the lcd cursor position to received next character
    }

  if(data_count == Password_Length-1)// if the array index is equal to the number of expected chars, compare data to master
  {
    lcd.clear();//clear lcd display
    if(attempt<=2) // for attempt less than or equal to 3
    {
    if(!strcmp(Data, CPass))//the received data is compared with password if it matches then it's success 
    {
      lcd.setCursor(5,0); //col:5;row:0
      lcd.print("WELCOME!");
      delay(1000);//wait for 1s
      lcd.clear();//clear lcd display
      lcd.setCursor(4,0); //col:4;row:0
      digitalWrite(5, HIGH); //the green led glows
      ServoOpen();//calling the void ServoOPen() function
      lcd.print("Door Open");
      delay(10000);//wait for 10s
      lcd.clear();//clear lcd display
      lcd.setCursor(2,0); //col:2;row:0
      lcd.print("Door Closing...");
      delay(5000);//wait for 5s
      digitalWrite(5, LOW); //the green led goes off
     ServoClose();//calling the void ServoClose() function
    }
    else //if the received data compared with password does'nt matches then access is denied 
    {
      lcd.setCursor(2,0); //col:2;row:0
      lcd.print("ACCESS DENIED");
      digitalWrite(4,HIGH);// the red led glows
      delay(1000);//wait for 1s
      digitalWrite(4,LOW);//the red led goes off
      delay(1000);//wait for 1s
     }
    attempt+=1;// if the attempts exceed 3 then the if loop will be terminated
    }
    else
    {
      //3 unsuccessfull attempts and thus raising alarm 
      while(attempt>=2)
     {
      lcd.clear();//clear lcd display
      lcd.setCursor(5,0); //col:5;row:0
      lcd.print("Failed");break;}
      if(strcmp(Data, MPass)){
      delay(1000);//wait for 1s
      lcd.clear();//clear lcd display
      clearData();//calling the function void clearData()
      digitalWrite(4,HIGH);//the red led glows
      delay(1000);//wait for 1s
      digitalWrite(3,HIGH);// the buzzer starts to beep
      tone(3,700);//the tone with which buzzer beeps}
      
      //deactivating alarm through master password
      if(!strcmp(Data, MPass)){
      lcd.clear();//clear lcd display
      delay(1000);//wait for 1s
      digitalWrite(4, LOW);     //the red led goes off         
      tone(3, 1000, 250);  //the tone with which buzzer beeps                           
      tone(3, 6000, 250);  //the tone with which buzzer beeps           
      delay(100);    //wait for 0.1s           
      tone(3, 1000, 250);           //the tone with which buzzer beeps                  
      tone(3, 6000, 250);          //the tone with which buzzer beeps   
      delay(100);//wait for 0.1s
      //after deactivating alarm and opening door through master password
      lcd.setCursor(5,0); //col:5;row:0
      lcd.print("WELCOME!");
      delay(1000);//wait for 1s
      //opening door
      digitalWrite(5, HIGH); //the green led glows
      delay(1000);//wait for 1s
      ServoOpen();//calling the function void ServoOpen()
      lcd.clear();//clear lcd display
      lcd.setCursor(4,0); //col:4;row:0
      lcd.print("Door open");
      delay(10000);//wait for 10s
      //closing door
      lcd.clear();//clear lcd display
      lcd.setCursor(2,0); //col:2;row:0
      lcd.print("Door closing....");
      delay(5000);//wait for 5s
      digitalWrite(5, LOW); // the green  led goes off
      ServoClose();//calling the function void ServoClose()
      attempt=1;
      }}
   lcd.clear();//clear lcd display
   clearData();//calling the function void clearData()
   }}

void clearData(){
  while(data_count !=0)
  {
  // This can be used for any array size,
   Data[data_count--] = 0; //clear array for new data
  }
}

  void ServoOpen()//for opening door through servo motor
{
  for (int pos = 180; pos >= 0; pos -= 5) { // goes from 0 degrees to 180 degrees
    // in steps of 1 degree
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
}

void ServoClose()//for closing door through servo motor
{
  for (int pos = 0; pos <= 180; pos += 5) { // goes from 180 degrees to 0 degrees
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
}                                                 
