
```cpp
#include <SoftwareSerial.h>    //شامل کردن کتابخانه های مورد نیاز
#include <LiquidCrystal_I2C.h>
#include <Keypad.h>


char ans;      // تعریف کردن متغیر های مورد نیاز برای کلیدها و متن به صورت گلوبال
String text;
char key;
String str;          

String Number = "+98"; //شماره مورد نظر با فرمت +98 است 

const byte ROW = 4;   // تعریف ابعاد و پین های مورد استفاده در کیپد
const byte COL = 3;
byte ROW_Pins[4] = {2,3,4,5};
byte COL_Pins[3] = {6,7,8};

char keymap[4][3] = {     // ساخت یک نقشه کاراکتری مورد استفاده در ساخت شی کیپد
  {'1','2','3'},
  {'4','5','6'},
  {'7','8','9'},
  {'*','0','#'}
};


String keyword[4][3] = {  // ساخت یک ماتریس از رشته ها برای ساخت یک مدل شبیه سازی شده از T9  مورد استفاده در گوشی های ساده
  {"abc","def","ghi"},
  {"jkl","mno","pqr"},
  {"stu","vwx","yz?"},
  {"*"  ,", !",  "#"}    
};

byte fullBlock[8] = {     // ساخت یک کاراکتر جدید یه مربع تو پر است برای ساخت یک انیمیشن در هنگام روشن شدن
  B11111, B11111, B11111, B11111,
  B11111, B11111, B11111, B11111
};

Keypad pad(makeKeymap(keymap), ROW_Pins, COL_Pins, ROW, COL);   // ساخت شی کیپد با استفاده از کلاس مربوط به کتابخانه 
LiquidCrystal_I2C lcd(0x27,16,2);                                                               // ساخت شی نمایشگر 
SoftwareSerial sim800l(10, 11);                     // ساخت یک درگاه ارتباط UART نرم افزاری برای ارتباط با sim800l

void SMS();



char   pre_key = 0;        
int    statue  = 0;

bool mp(char a, int &i, int &j) {      // ساخت یک تابع برای پیدا کردن رشته کلید فشرده شده در کیپد برای انتخاب کاراکتر
  if (a >= '1' && a <= '9') {
    int n = a - '1';
    i = n / 3;
    j = n % 3;
    return true;
  }
  if (a=='0'){i=3;j=1;return true;}
  return false; 
}

char waitKeyRaw() {                                                                         // ساخت یک تابع  برای دریافت کلید .
  char k = 0;                       // این تابع فراخوانی تابع ring  را برای چک کردن زنگ خور را ندارد تا در داخل ان استفاده شود
  while (!k) {
    k = pad.getKey();
    delay(20);
  }
  return k;
}

char waitKey() {            //ساخت تابع برای دریافت کلید با فراخوانی تابع زنگ خور. چون برنامه تقریبا در اکثر زمان اجرا 
  char k = 0;                //منتظر دریافت کلید میماند و توابع دریافت کلید برنامه را متوقف میکنند باید در اینجا چک را انجام داد
  while (!k) {
    Ring();                   
    k = pad.getKey();
    delay(20);
  }
  return k;
}


void refreshLCD() {       // ساخت یک تابع برای اینکه اگر رشته ورودی از 16 بیشتر شد شیفت انجام شود تا اخر متن نمایش داده شود
  lcd.setCursor(0,1);
  lcd.print("                "); 
  lcd.setCursor(0,1);
  if (str.length() <= 16) {
    lcd.print(str);
  } else {
    lcd.print(str.substring(str.length() - 16));
  }
}



bool handleT9(char k) {       // ساخت تابعی برای مدیریت کلید ها در کیبورد و انتخاب کاراکتر از بین رشته ها
  if (k == '#') return true;   

  if (k == '*') {
    // Backspace
    if (str.length() > 0) str.remove(str.length() - 1);
    pre_key = 0;
    statue  = 0;
    refreshLCD();
    return false;
  }

  int i, j;
  if (!mp(k, i, j)) return false;

  if (k == pre_key && str.length() > 0) {
    statue = (statue + 1) % (int)keyword[i][j].length();
    str.remove(str.length() - 1);
  } else {
    statue = 0;
  }

  str += keyword[i][j][statue];
  pre_key = k;
  refreshLCD();
  return false;

}
void Get_Text() {    //دریافت کلید و فراخوانی تابع هندل کننده برای  متن نهایی پیامک
  str     = "";
  pre_key = 0;
  statue  = 0;

  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Enter text:");
  refreshLCD();

  while (true) {
    char k = waitKey();
    if (handleT9(k)) break;  
  }


  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Text OK:");
  lcd.setCursor(0,1);
  lcd.print(str.substring(0, min((int)str.length(), 16)));
  delay(1500);
}


void setup() {     // تابع ستاپ برای اینترو و انجام تنضیمات مورد نیاز
  Serial.begin(9600);
  sim800l.begin(9600);

  sim800l.println("AT");
  delay(500);
  while (sim800l.available()) Serial.write(sim800l.read());

  lcd.init();
  lcd.backlight();
  lcd.createChar(1, fullBlock);
  lcd.clear();

  for (int i = 0; i < 16; i++) {
    lcd.setCursor(i,0);    lcd.write(byte(1));
    lcd.setCursor(15-i,1); lcd.write(byte(1));
    delay(70);
    lcd.clear();
  }
  for (int i = 0; i < 16; i++) {
    lcd.setCursor(15-i,0); lcd.write(byte(1));
    lcd.setCursor(i,1);    lcd.write(byte(1));
    delay(70);
    lcd.clear();
  }

  lcd.setCursor(0,0);
  lcd.print("Initializing...");
  Serial.println("Initializing...");
  delay(2000);
}


void loop() {   // تابع لوپ و فراخوانی تابع شروع 
  Start();
}


void updateSerial() {                       // ایت تابع  برای ارتباط سریال مستقیم ساخته شده و البته فعلا کاربردی ندارد
  while (!(Serial.available())) {}
  if (Serial.available()) {
    sim800l.write(Serial.read());
  }
  while (sim800l.available()) {
    Serial.write(sim800l.read());
  }
  Serial.print("1) Continue  ");
  Serial.println("2) Back ");
  while (!(Serial.available())) {}
  ans = Serial.read();
  if (ans == '1') {
    updateSerial();
    while (Serial.available()) Serial.read();
  }
  if (ans == '2') {
    Start();
    while (Serial.available()) Serial.read();
  }
}


void Start() {                    // ساخت اولین منو برای دریافت دستورات و فراخوانی توابع مورد نیاز
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("1) AT    2) SMS");
  lcd.setCursor(0,1);
  lcd.print("3) Call  4) Calc");
  Serial.print("1) AT  2) SMS ");
  Serial.println("3) Call  4) Calc");
  char k = waitKey();
  if (k == '1') updateSerial();
  if (k == '2') two();
  if (k == '3') Call();
}

void two(){lcd.clear();   // منوی مربوط به بعد از انتخاب گزینه 2 در منوی اول
  lcd.setCursor(0,0);
  lcd.print(" 1)Send  2)Read");
  char k = waitKey();
  if (k == '1') SMS();
  else if (k == '2') Read();
  else {two();}
  }

void Read() {   // خواند اخرین چیامک موجود در حافظ هsim800  و نمایش و سپس خالی کردن حافظه
  
  while (Serial.available()) Serial.read();
  while (sim800l.available()) sim800l.read();

  
  sim800l.println("AT+CMGF=1");
  delay(500);
  while (sim800l.available()) Serial.write(sim800l.read());

  // خواندن پیام 1
  sim800l.println("AT+CMGR=1");
  
  String response = "";
  unsigned long t = millis();
  while (millis() - t < 3000) {
    while (sim800l.available()) {
      char c = sim800l.read();
      response += c;
      Serial.write(c);
    }
  }

  Serial.println();
  Serial.println("---- RAW RESPONSE ----");
  Serial.println(response);
  Serial.println("----------------------");

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Msg:");

 
  String body = "";
  int idx = response.indexOf("+CMGR:");  // چون sim800  پاسخ را در چند خط به همراه اطلاعات اضافی میدهد در اینجا منت مورد نیاز استخراج میشود
  if (idx >= 0) {
  
    int nl1 = response.indexOf('\n', idx);
    if (nl1 >= 0) {

      int start = nl1 + 1;
      int end = response.indexOf('\n', start);
      if (end < 0) end = response.length();
      
      body = response.substring(start, end);
      body.trim();

      body.replace("\r", "");
    }
  }

  lcd.setCursor(0, 1);
  if (body.length() > 0) {
    lcd.print(body.substring(0, 16));
    Serial.print("Body: ");
    Serial.println(body);
  } else {
    lcd.print("No message");
    Serial.println("No message parsed");
  }

  delay(3000);


  sim800l.println("AT+CMGD=1,4");
  delay(500);
  while (sim800l.available()) Serial.write(sim800l.read());

  Start();
}

void SMS()  {          // ساخت تابعی برای تنظیم دستگاه روی حالت متنی سپس دریافت شماره مورد نظر و سپس متن و در نهایت ارسال
  delay(100);
  while (Serial.available()) Serial.read();

  sim800l.println("AT");
  delay(500);
  while (sim800l.available()) Serial.write(sim800l.read());

  sim800l.println("AT+CMGF=1");
  delay(500);
  while (sim800l.available()) Serial.write(sim800l.read());

  get_number();
  String k = "AT+CMGS=\"";
  k += Number;
  k+= "\"";
  sim800l.println(k);
  delay(1000);
  while (sim800l.available()) Serial.write(sim800l.read());

  Serial.println("Enter your text:");
  Get_Text();  

  delay(100);
  sim800l.print(str);
  delay(500);
  sim800l.write(26);  
  delay(500);
  while (sim800l.available()) Serial.write(sim800l.read());

  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Sending...");
  delay(1500);
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Sent");
  delay(1000);
  Ring();
  Start();
}

void get_number(){      // تابعی برای دریافت شماره تلفن از کاربر مورد استفاده در توابع پیام و زنگ
  Ring();
  Number = "+98";
  char gnum;
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("enter phone num:");lcd.setCursor(0,1);lcd.print(Number);
  while (true){
    gnum = waitKey();
    if (gnum == '*'){if (Number.length() > 0) Number.remove(Number.length() - 1);}
    if (gnum == '#'){break;}
    else {Number += gnum;lcd.setCursor(0,1);lcd.print(Number);}
    }
  }

void Call(){       // ساخت تابع برای شروع تماس 
  get_number();
  sim800l.println("AT");
  delay(500);
  while (sim800l.available()) Serial.write(sim800l.read());
  String ki = "ATD";
  ki += Number;
  ki += ";";
  sim800l.println(ki);
  lcd.clear();
  lcd.setCursor(1,0);
  lcd.print(Number);
  lcd.setCursor(3,1);
  lcd.print("1) end call");
  char gnum;
  gnum = waitKeyRaw();
  if (gnum == '1'){sim800l.println("ATH");
  
  }
  
  Start();
  
  }

 void Ring() {     // در این تابع ارتباط سریال بررسی میشود و اگر sim800  کلمه ring را ارسال کند یعنی در حال زنگ خوردن است و منوی مورد نظر نمایش داده میشود
  if (!sim800l.available()) return;

  String response = sim800l.readStringUntil('\n');
  response.trim();
  Serial.println(response);

  if (response == "RING") {
    lcd.clear();
    lcd.setCursor(1,0);
    lcd.print("phone ringing");
    lcd.setCursor(1,1);
    lcd.print("1)ANS  2)REJECT");

    char l = waitKeyRaw();     
    if (l == '1') {
      sim800l.println("ATA");
      lcd.setCursor(1,1);
      lcd.print("                  ");
      lcd.setCursor(5,1);
      lcd.print("1)END");
      l = waitKeyRaw();
      if (l == '1') sim800l.println("ATH");
    } else {
      sim800l.println("ATH");
    }
    lcd.clear();
    while (sim800l.available()) sim800l.read();
  Start();
  }

}
```
