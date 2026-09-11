## Creating our own class for keypad
The logic works as follows: we have a set of pins acting as rows that are driven high, and another set acting as columns that are pulled up.
In each cycle, we check which column pin has gone LOW; that is the pin corresponding to the pressed key.

### stage one
In the first stage, we write this code for a specific, common keypad—such as a 4x3 model—and then upgrade it.

```cpp
int row0 = 2;
int row1 = 3;
int row2 = 4;
int row3 = 5;

int col0 = 6;
int col1 = 7;
int col2 = 8;

char keymap[4][3] = {{'1','2','3'},
                     {'4','5','6'},
                     {'7','8','9'},
                     {'*','0','#'}};

void setup() {

  Serial.begin(9600);

  pinMode(row0, OUTPUT);
  pinMode(row1, OUTPUT);
  pinMode(row2, OUTPUT);
  pinMode(row3, OUTPUT);
  
  digitalWrite(row0,HIGH);
  digitalWrite(row1,HIGH);
  digitalWrite(row2,HIGH);
  digitalWrite(row3,HIGH);

  pinMode(col0,INPUT_PULLUP);
  pinMode(col1,INPUT_PULLUP);
  pinMode(col2,INPUT_PULLUP);

}

void loop() {
  
  int row_list[4] = {row0,row1,row2,row3};
  int col_list[3] = {col0,col1,col2};
  
  for(int i = 0; i<4; i++){
    
    digitalWrite(row_list[i],LOW);
    
    for (int j = 0; j<3; j++){
      int reading = digitalRead(col_list[j]);
      
      if (reading == LOW){
        Serial.println(keymap[i][j]);
      }
      
    }
    
    digitalWrite(row_list[i],HIGH);
  }
  delay(10);
}
```
### stage two
if you check the previous stages code, you will see that with pressing keys, it will print many times in the monitor. 

```cpp
int row = 4;
int col = 3;

int rows[4] = {2, 3, 4, 5};
int cols[3] = {6, 7, 8};

char keymap[4][3] = {{'1','2','3'},
                     {'4','5','6'},
                     {'7','8','9'},
                     {'*','0','#'}};

class key_pad {
  private:
    char (*key_map)[3];
    int* row_pins;
    int* col_pins;
    int  row;
    int  col;
    char key = '\0';
    char fkey = '\0';
    bool pressed = false;

  public:
    key_pad(char (*km)[3] , int* a , int* b , int c , int d){
      key_map = km;
      row_pins = a;
      col_pins = b;
      row = c;
      col = d;
      }
    void Begin(){
      for (int i = 0 ; i<row ; i++){
        pinMode(row_pins[i],OUTPUT);
        }
      delay(15);
      for (int i = 0 ; i<row ; i++){
        digitalWrite(row_pins[i],HIGH);
        }
      for (int i = 0 ; i<col ; i++){
         pinMode(col_pins[i],INPUT_PULLUP);
      }
      
      }
    void set_key(){
       fkey = '\0';
      for(int i = 0; i<4; i++){
    
         digitalWrite(row_pins[i],LOW);
       
    
         for (int j = 0; j<3; j++){
           int reading = digitalRead(col_pins[j]);
      
            if ((reading == LOW)){
             fkey = key_map[i][j];
         
            }
      
          }
          digitalWrite(row_pins[i],HIGH);
        }
        if (fkey != '\0' && !pressed) {
    key = fkey;       
    pressed = true;
  }
  else if (fkey == '\0') {
    pressed = false;  
  }
  else {
    key = '\0';     
  }
    }

    char get_key(){
      delay(10);
      return key;
      
      }
    
  };

key_pad mypad(keymap,rows,cols,row,col);

void setup() {
  Serial.begin(9600);
  mypad.Begin();
}


void loop() {
  mypad.set_key();
  char ne = mypad.get_key();
  if(ne != '\0'){
    Serial.println(ne);
    }
  
  }

```
now its ready to use. not easy like the real keypad library, but the important point is knowing its logic.











