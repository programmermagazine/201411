## 函數指標陣列 (Array of Function Pointers) (作者：研發養成所 Bridan)

函數指標陣列，這是一種 C/C++ 程式語言的高階設計技巧，希望能有較高的執行效能。
我以 Arduino 當作測試平台，比較兩種程式設計技巧，發現與我的認知有些差異。

先看傳統設計方式，用 switch case 執行不同功能：

```CPP
//
// Author: Bridan
//         http://4rdp.blogspot.com
// Date:   2014/09/27
//
// Brief:  Test switch case
//

void setup() {
  Serial.begin(9600);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for Leonardo only
  }

  TCCR1A = 0x00;                // Normal mode, just as a Timer
  TCCR1B &= ~_BV(CS12);         // no prescaling
  TCCR1B &= ~_BV(CS11);      
  TCCR1B |= _BV(CS10);   
}

void loop() {
  byte i;

  TCNT1 = 0;     // reset timer
  for (i=0 ; i＜3 ; i++) {
    switch (i) {
      case 0:
        Serial.println("CASE 0");
        break;
      case 1:
        Serial.println("CASE 1");
        break;
      case 2:
        Serial.println("CASE 2");
        break;
    }
  }
  Serial.println(TCNT1);
}
```

switch case 3 個時，編譯 2410 bytes，執行 6092 ~ 6100 timer clock
switch case 4 個時，編譯 2430 bytes，執行 8136 ~ 8146 timer clock
switch case 5 個時，編譯 2458 bytes，執行 10185 ~ 10195 timer clock

將上面程式修改成函數指標陣列，以查表方式直接跳到執行的程式：

```CPP
//
// Author: Bridan
//         http://4rdp.blogspot.com
// Date:   2014/09/27
//
// Brief:  Test Array of Function Pointers
//

void setup() {
  Serial.begin(9600);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for Leonardo only
  }

  TCCR1A = 0x00;                // Normal mode, just as a Timer
  TCCR1B &= ~_BV(CS12);         // no prescaling
  TCCR1B &= ~_BV(CS11);      
  TCCR1B |= _BV(CS10);   
}

void FUNC0(void) {
     Serial.println("CASE 0");
}
void FUNC1(void) {
     Serial.println("CASE 1");
}
void FUNC2(void) {
     Serial.println("CASE 2");
}

void (*TABLE_JUMP[])(void) = {
  FUNC0,
  FUNC1,
  FUNC2
};

void loop() {
  byte i;

  TCNT1 = 0;     // reset timer
  for (i=0 ; i＜3 ; i++) {
    (*TABLE_JUMP[i])();
  }
  Serial.println(TCNT1);
}
```

TABLE 3 個時，編譯 2438 bytes，執行 6082 ~ 6096 timer clock
TABLE 4 個時，編譯 2470 bytes，執行 8130 ~ 8142 timer clock
TABLE 5 個時，編譯 2504 bytes，執行 10176 ~ 10194 timer clock

以往我所用過的 compiler，switch case 相當於很多 if ... else ... 的組合，條件一個一個比較，數值越大的條件，花費比較的時間越多，以上面的例子在比較方面所費的時間 = 1 + 2 + 3 + ... + N，而函數指標陣列查表時間約 = 1 x N，比 switch case 有效率，這部分與結果相符 (比較條件太少，不易看出差異)。

至於程式碼大小，發現越多條件狀況，以函數指標陣列方式設計比 switch case 程式碼多？因為不清楚 Arduino compiler 如何設計，無法進一步評論，但直覺 Arduino compiler 缺少這方面最佳化處理。

