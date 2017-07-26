#include<SoftwareSerial.h> // BLE
#include <Wire.h> // RTC 데이터핀 연결
// SCL - pin A5
// SDA - pin A4

#define DS3231_I2C_ADDRESS 104 // RTC
#define MSG_SIZE 17 //<-- YY/MM/DD'deli'HH:MM:SS
#define PIN_BLUETOOTH_RX 12
#define PIN_BLUETOOTH_TX 13

SoftwareSerial bluetooth(PIN_BLUETOOTH_TX, PIN_BLUETOOTH_RX); // BLE

byte seconds, minutes, hours, day, date, month, year; // 시간 (2진수)
char weekDay[4]; // 요일 문자열

// AtMega 328 is Little Endian
byte tMSB, tLSB; // MostSignificantBit, LeastSignificantBit 
float temp3231;

int cig_num = -1; // 남은 담배 개수
char message[MSG_SIZE]; // 메시지
byte delimeter = '-'; // 날짜-시간
byte date_deli = '/'; // 날짜는 YY/MM/DD
byte time_deli = ':'; // 시간은 HH:MM:SS

boolean isTimeSet; // 시간 동기화 여부

/* Eject pin debouncing data structure */
typedef struct {
  int pinNum;
  int isTouched = HIGH;
  int state;
  int lastState = LOW;
  long debounceDelay = 50;
  long lastDebounceTime = 0; // 가장최근 닿았던 시각
  int reading = 0;
} Eject;

Eject eject;

void setup() {
  Wire.begin(); // RTC setup
  Serial.begin(9600); // Console setup
  bluetooth.begin(9600); // BLE setup

  // Eject pin setup
  eject.pinNum = 2;
  pinMode(eject.pinNum, INPUT_PULLUP);

  // 시간 동기화 여부
  isTimeSet = false;
}

void loop() {

  watchConsole(); // 시간정보 설정
  get3231Date(); // 시간정보 출력
    
  //if(isTimeSet == false) {
    Serial.print(weekDay);
    Serial.print(", 20");
    Serial.print(year, DEC);
    Serial.print("/");
    Serial.print(month, DEC);
    Serial.print("/");
    Serial.print(date, DEC);
    Serial.print(" - ");
    Serial.print(hours, DEC); 
    Serial.print(":"); 
    Serial.print(minutes, DEC); 
    Serial.print(":"); 
    Serial.print(seconds, DEC);
    Serial.print(" - Temp: "); 
    Serial.println(get3231Temp());
//    delay(1000);
  //}
  
  ejectCommand();
}


// 10진수를 2진화 10진수인 BCD 로 변환 (Binary Coded Decimal)
byte decToBcd(byte val) {
  return ( (val/10*16) + (val%10) );
}

// Console에서 T(시간설정명령) 받는 watcher
// set3231Date()를 수행
// 추후, if(bluetooth.available()) { if(bluetooth.read() == 84) { set3231Date(); } } 로 바꿀 것
void watchConsole() {
//  if (Serial.available() || bluetooth.available()) {      // Look for char in serial queue and process if found
//    if (Serial.read() == 84 || bluetooth.read() == 84) {   //If command = "T" Set Date
//      set3231Date();
//      get3231Date();
//      isTimeSet = true; // 더이상 동기화 x
//      Serial.println(" ");
//    }
//  }
//  if(Serial.available()){
//    if(Serial.read() == 84){
//      set3231Date();
//      get3231Date();
//      Serial.println(" ");
//    }
//  }
  if(Serial.available() && Serial.read() == 84 ) {
    set3231DateBySerial();
    Serial.println("Time Synced By SerialMonitor! ");
  }
  if(bluetooth.available() && bluetooth.read() == 84 ) {
    set3231DateByBluetooth();
    Serial.println("Time Synced By Bluetooth! ");
  }
  get3231Date();
  Serial.println(" ");
}


//시간설정
// T(설정명령) + 년(00~99) + 월(01~12) + 일(01~31) + 시(00~23) + 분(00~59) + 초(00~59) + 요일(1~7, 일1 월2 화3 수4 목5 금6 토7)
// 예: T1605091300002 (2016년 5월 9일 13시 00분 00초 월요일)
void set3231DateBySerial() {
  year    = (byte) ((Serial.read() - 48) *10 +  (Serial.read() - 48));
  month   = (byte) ((Serial.read() - 48) *10 +  (Serial.read() - 48));
  date    = (byte) ((Serial.read() - 48) *10 +  (Serial.read() - 48));
  hours   = (byte) ((Serial.read() - 48) *10 +  (Serial.read() - 48));
  minutes = (byte) ((Serial.read() - 48) *10 +  (Serial.read() - 48));
  seconds = (byte) ((Serial.read() - 48) * 10 + (Serial.read() - 48));
  day     = (byte) (Serial.read() - 48);
 
  Wire.beginTransmission(DS3231_I2C_ADDRESS);
  Wire.write(0x00);
  Wire.write(decToBcd(seconds));
  Wire.write(decToBcd(minutes));
  Wire.write(decToBcd(hours));
  Wire.write(decToBcd(day));
  Wire.write(decToBcd(date));
  Wire.write(decToBcd(month));
  Wire.write(decToBcd(year));
  Wire.endTransmission();
}

void set3231DateByBluetooth() {
 
  year    = (byte) ((bluetooth.read() - 48) *10 +  (bluetooth.read() - 48));
  month   = (byte) ((bluetooth.read() - 48) *10 +  (bluetooth.read() - 48));
  date    = (byte) ((bluetooth.read() - 48) *10 +  (bluetooth.read() - 48));
  hours   = (byte) ((bluetooth.read() - 48) *10 +  (bluetooth.read() - 48));
  minutes = (byte) ((bluetooth.read() - 48) *10 +  (bluetooth.read() - 48));
  seconds = (byte) ((bluetooth.read() - 48) * 10 + (bluetooth.read() - 48));
  day     = (byte) (bluetooth.read() - 48);
 
  Wire.beginTransmission(DS3231_I2C_ADDRESS);
  Wire.write(0x00);
  Wire.write(decToBcd(seconds));
  Wire.write(decToBcd(minutes));
  Wire.write(decToBcd(hours));
  Wire.write(decToBcd(day));
  Wire.write(decToBcd(date));
  Wire.write(decToBcd(month));
  Wire.write(decToBcd(year));
  Wire.endTransmission();
}

void get3231Date() {
  // send request to receive data starting at register 0
  Wire.beginTransmission(DS3231_I2C_ADDRESS); // 104 is DS3231 device address
  Wire.write(0x00); // start at register 0
  Wire.endTransmission();
  Wire.requestFrom(DS3231_I2C_ADDRESS, 7); // request seven bytes
 
  if(Wire.available()) {
    seconds = Wire.read(); // get seconds
    minutes = Wire.read(); // get minutes
    hours   = Wire.read();   // get hours
    day     = Wire.read();
    date    = Wire.read();
    month   = Wire.read(); //temp month
    year    = Wire.read();
       
    seconds = (((seconds & B11110000)>>4)*10 + (seconds & B00001111)); // convert BCD to decimal
    minutes = (((minutes & B11110000)>>4)*10 + (minutes & B00001111)); // convert BCD to decimal
    hours   = (((hours & B00110000)>>4)*10 + (hours & B00001111)); // convert BCD to decimal (assume 24 hour mode)
    day     = (day & B00000111); // 1-7
    date    = (((date & B00110000)>>4)*10 + (date & B00001111)); // 1-31
    month   = (((month & B00010000)>>4)*10 + (month & B00001111)); //msb7 is century overflow
    year    = (((year & B11110000)>>4)*10 + (year & B00001111));
  }
  else {
    //oh noes, no data!
  }
 
  switch (day) {
    case 1:
      strcpy(weekDay, "Sun");
      break;
    case 2:
      strcpy(weekDay, "Mon");
      break;
    case 3:
      strcpy(weekDay, "Tue");
      break;
    case 4:
      strcpy(weekDay, "Wed");
      break;
    case 5:
      strcpy(weekDay, "Thu");
      break;
    case 6:
      strcpy(weekDay, "Fri");
      break;
    case 7:
      strcpy(weekDay, "Sat");
      break;
  }
}

float get3231Temp() {
  //temp registers (11h-12h) get updated automatically every 64s
  Wire.beginTransmission(DS3231_I2C_ADDRESS);
  Wire.write(0x11);
  Wire.endTransmission();
  Wire.requestFrom(DS3231_I2C_ADDRESS, 2);
 
  if(Wire.available()) {
    tMSB = Wire.read(); //2's complement int portion
    tLSB = Wire.read(); //fraction portion
   
    temp3231 = (tMSB & B01111111); //do 2's math on Tmsb
    temp3231 += ( (tLSB >> 6) * 0.25 ); //only care about bits 7 & 8
  }
  else {
    //error! no data!
  }
  return temp3231;
}


void ejectCommand() {
  /* 1. notifier의 상태를 읽음 */
  eject.reading = digitalRead(eject.pinNum);

  /* 2. 5V와 notifier가 닿아 state에 변화가 생겼을 때 lastDebounceTime 설정 
   * 즉, 맞닿지 않을 경우 lastDebounceTime은 최신화 되지 않음 */
  if(eject.reading != eject.lastState) {
    eject.lastDebounceTime = millis();
  }

  /* 3. 5V와 notifier가 닿은 시간이 debounceDelay보다 길며
   * 즉, 바로 맞닿고 debounceDelay 시간이 지나기 전에는 이 조건문을 거치지 않음 */
  if( (millis() - eject.lastDebounceTime) > eject.debounceDelay ) {

    /* 4. 5V와 notifier가 닿아 변화가 있을 때
     * 즉, 맞닿은 후 또는 떨어진 후 변화가 없으면 이 조건문을 거치지 않음 */
    if( eject.reading != eject.state ) {
      eject.state = eject.reading;

      /* 5. 5V와 notifier가 닿아서 발생한 것 이라면
       * 즉, 5V와 notifier가 떨어질때 발생한 것은 이 조건문을 거치지 않음 */
      if( eject.state == HIGH ) {
        eject.isTouched = !eject.isTouched;
        
        /* 이 곳에 코드를 작성!! */
        Serial.println("Touched!");
        setMessage(message); // 메시지 설정
        sendMessageToMobile(getMessage()); // 메시지 전송
        printMessageToConsole(getMessage()); // 메시지 콘솔에 출력
        
      }
    }
  }
  /* 6. 5V와 notifier가 닿아서 발생한 state변화가 한번 이루어진 후, lastState를 바꿔준다
   * 5V와 notifier가 닿고 debounceDelay보다 시간이 짧다면 위 조건문 진행 없이 아래 명령 수행 */
  eject.lastState = eject.reading;
}

void sendMessageToMobile(char *msg){
  bluetooth.print(msg);
}

void printMessageToConsole(char *msg){
  Serial.println(msg);
}

void recvMessageFromMobile(){
   // watchConsole, set3231Date함수 참고해 안드로이드 동기화버튼 누를시 안드로이드의 시간 받아오는 함수
}

void setMessage(char* msg) { // <-- YY/MM/DD'deli'HH:MM:SS
  sprintf(msg, "%d%c%d%c%d%c%d%c%d%c%d"
  , year, date_deli, month, date_deli, date, delimeter, hours, time_deli, minutes, time_deli, seconds);
}

char* getMessage() {
  return message;
}

//void ejectCommand(Eject eject) {
//  /* 1. notifier의 상태를 읽음 */
//  eject.reading = digitalRead(eject.pinNum);
//
//  /* 2. 5V와 notifier가 닿아 state에 변화가 생겼을 때 lastDebounceTime 설정 
//   * 즉, 맞닿지 않을 경우 lastDebounceTime은 최신화 되지 않음 */
//  if(eject.reading != eject.lastState) {
//    eject.lastDebounceTime = millis();
//  }
//
//  /* 3. 5V와 notifier가 닿은 시간이 debounceDelay보다 길며
//   * 즉, 바로 맞닿고 debounceDelay 시간이 지나기 전에는 이 조건문을 거치지 않음 */
//  if( (millis() - eject.lastDebounceTime) > eject.debounceDelay ) {
//    Serial.print("*");
//
//    /* 4. 5V와 notifier가 닿아 변화가 있을 때
//     * 즉, 맞닿은 후 또는 떨어진 후 변화가 없으면 이 조건문을 거치지 않음 */
//    if( eject.reading != eject.state ) {
//      eject.state = eject.reading;
//      Serial.println("//");
//
//      /* 5. 5V와 notifier가 닿아서 발생한 것 이라면
//       * 즉, 5V와 notifier가 떨어질때 발생한 것은 이 조건문을 거치지 않음 */
//      if( eject.state == HIGH ) {
//        eject.isTouched = !eject.isTouched;
//        Serial.println("///");
//        
//        /* 이 곳에 코드를 작성!! */
//        Serial.println("Touched!");
//        setMessage(message); // 메시지 설정
//        sendMessageToMobile(getMessage()); // 메시지 전송
//        
//        // EEPROM 추가해야함
//        
//      }
//    }
//  }
//  /* 6. 5V와 notifier가 닿아서 발생한 state변화가 한번 이루어진 후, lastState를 바꿔준다
//   * 5V와 notifier가 닿고 debounceDelay보다 시간이 짧다면 위 조건문 진행 없이 아래 명령 수행 */
//  eject.lastState = eject.reading;
//}
