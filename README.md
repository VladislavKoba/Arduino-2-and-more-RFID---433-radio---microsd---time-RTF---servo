# Arduino-2-and-more-RFID---433-radio---microsd---time-RTF---servo
Arduino Nano v3 project for security system
-------
code whith comments here becouse nano v3 has not much memory
-------
This project allows to make security system on its base. U can regist the number of used rfid card, the time when it was used, the rfid module, that regist the card. It understand when the door is closing, and it can be opened on a distance using radio.

The main problem that i faced whith was the ss pins on rfid modules, there is should be only one ss pin conected to arduino, in other case, arduino just dont react on 2 and more modules. And using of Bread board(model shield) doesn`t helps, couse rfid modules wants quality voltage, so, the decision was or soldering, or creating of a board. And, as i don't know how to etch boards, i decided to create this monster only by soldering:D  
![alt text](https://github.com/VladislavKoba/Arduino-2-and-more-RFID---433-radio---microsd---time-RTF---servo/blob/master/photo_2018-04-23_16-48-35.jpg?raw=true)
```
#include <SPI.h>
#include <MFRC522.h>
#include <iarduino_RTC.h>
#include <SD.h>
#include <Servo.h>


#define RST_PIN    9   // 
#define SS_PIN    10    //

Servo myservo;
MFRC522 mfrc522ent(SS_PIN, RST_PIN);
MFRC522 mfrc522ext(7, RST_PIN);
iarduino_RTC time(RTC_DS1307);
File myFile;
int val = 0;
int yal = 0;

void setup() {
  
  
  
  Serial.begin(9600);    
  while (!Serial);    
  SPI.begin();      
  mfrc522ent.PCD_Init();  
  mfrc522ext.PCD_Init();
  pinMode(3, OUTPUT);
  pinMode(8, OUTPUT);
  pinMode(2, INPUT); 
  pinMode(5, INPUT);
  //ShowReaderDetails();  
  Serial.println(F("Scan PICC to see UID, type, and data blocks..."));
  Serial.print("Initializing SD card...");

  if (!SD.begin(4)) {
    Serial.println("initialization failed!");
    return;
  }
  Serial.println("initialization done.");
  time.begin();
}

void loop() {
  val = digitalRead(2);//reads if there is a signal on a radio 433 module
  yal = digitalRead(5);//reads if the reed switch(girkon) is closed
  //array of allowed rfid cards 
  byte uidCard[2][4] ={ {136, 173, 73, 0},
                        {115, 242, 19, 0}
  };

 


  //open servo if there is a signal on radio module and servo is closed
  if (val == HIGH) {
    digitalWrite (3, HIGH);
    if(myservo.read() != 0){
    myservo.attach(6);
    myservo.write(0);
    delay(1000);
    myservo.detach();}
 } else {
  digitalWrite(3, LOW); 
  }
  
  //close servo if reed swith closed and servo is open
  if (yal == HIGH) {
    digitalWrite (8, HIGH);
    if(myservo.read() != 180){
    delay(2000);
    myservo.attach(6);
    myservo.write(180);
    delay(1000);
    myservo.detach();}
 } else {
  digitalWrite(8, LOW); 
  }


  
    if(mfrc522ent.PICC_IsNewCardPresent()){
    if(mfrc522ent.PICC_ReadCardSerial()){
    //match rfid card with array of allowed, if there is one, it opens servo and write on microsd number of card and curent time 
    for (byte j=0; j<2; j++){
      if(uidCard[j][0] == mfrc522ent.uid.uidByte[1] && uidCard[j][1] == mfrc522ent.uid.uidByte[2] && uidCard[j][2] == mfrc522ent.uid.uidByte[3] && uidCard[j][3] == mfrc522ent.uid.uidByte[4]){
        Serial.println("entery");
        for (byte i=0; i<4; i++){Serial.println(uidCard[j][i]);}
        Serial.println(time.gettime("d-m-Y, H:i:s, D"));
        
        if(myservo.read() != 0){
        myservo.attach(6);
        myservo.write(0);
        delay(1000);
        myservo.detach();}
        
        myFile = SD.open("t.txt", FILE_WRITE);

          // if the file opened okay, write to it
          if (myFile) {
            Serial.print("Writing to test.txt...");
            myFile.print("enter ");
            for (byte i=0; i<4; i++){myFile.print(uidCard[j][i]);myFile.print(" ");}
            myFile.print(time.gettime("dmY His"));
            myFile.println(" ");
            // close the file
            myFile.close();
            Serial.println("done.");
            digitalWrite(3, HIGH);
          } else {
            // if the file didn't open, print an error
            Serial.println("error opening test.txt");
          }
     }}
    }}
     
    if( mfrc522ext.PICC_IsNewCardPresent()){
    if( mfrc522ext.PICC_ReadCardSerial()){
     for (byte j=0; j<2; j++){
      if(uidCard[j][0] == mfrc522ext.uid.uidByte[1] && uidCard[j][1] == mfrc522ext.uid.uidByte[2] && uidCard[j][2] == mfrc522ext.uid.uidByte[3] && uidCard[j][3] == mfrc522ext.uid.uidByte[4]){
        Serial.println("exit");
        Serial.println(time.gettime("d-m-Y, H:i:s, D"));
        
        if(myservo.read() != 0){
        myservo.attach(6);
        myservo.write(0);
        delay(1000);
        myservo.detach();}
        
        myFile = SD.open("t.txt", FILE_WRITE);
         if (myFile) {
            Serial.print("Writing to test.txt...");
            myFile.print("exit ");
            for (byte i=0; i<4; i++){myFile.print(uidCard[j][i]);myFile.print(" ");}
            myFile.print(time.gettime("dmY His"));
            myFile.println(" ");
            myFile.close();
            Serial.println("done.");
            digitalWrite(3, HIGH);
          } else {
            Serial.println("error opening test.txt");
          }
     }}}}

    
   
    delay(1000);
    
}
```
