### usin library
using this library is very simple. we got a few commands for control.<br>
each I2C module has got an adrres wich we have to know our lcd's adrres. because when we want to create object for that, we need to enter that adrres. 
### making obj
```cpp
#include <liquidCristal_I2C.h>
liquidCristal_I2C lcd(adrres,col,row);
```
### commands
```cpp
lcd.init();
```
this  is for starting comminucation like begin function in UART.
```cpp
lcd.clear();
```
it clears the screan.
```cpp
lcd.setCorsur(a,b);
```
we can set our corsur with this function. the text will start from our corsur.
```cpp
lcd.print();
```
we write with this function.
<br> we don't need any more in this library.
