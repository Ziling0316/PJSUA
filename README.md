# 變聲器
## 用途
- 使用者可選擇想要的聲音做相對的頻率改變，改變使用者說話的聲音
## 注意事項
- 因為我們的變聲器在iLBC才有用，所以打電話的人要把 iLBC 優先度調到最高

## 使用步驟
1. 使用者輸入聲音選項到frequency_index變數裡面後進行程式的重新編譯
   - `0` : 正常聲音
   - `1` : 更低沉聲音
   - `2` : 低沉聲音
   - `3` : 高亢聲音
   - `4` : 更高亢聲音
3. 按下m並且輸入通話對象的IP address及port number
4. 即可用變化後的聲音開始通話

## 程式說明(額外增加的部分)

- 修改檔案：encode.c
- 使用者將輸入的聲音選項儲存到以下global variable當中，default值為1

```cpp
int frequency_index = 1
```

- d指向修改過頻率的聲音，並將其內容覆蓋掉原本存放聲音的data array的內容
- bass則為更改成低頻頻率時所要乘的倍數

```cpp
float* d
int bass
```

- 根據frequency_index的值做bass跟d的變更
    - 低頻就是將波長拉長使得頻率變低，故需要有bass，且d會變成原來data的bass倍
    - 高頻就是將波長縮短使得頻率變高，不需要有bass，且d會變成原來data的1/frequency_index倍

```cpp
switch (frequency_index) {
   case -2:  /*更低沉聲音*/
       bass = 3;
       d = malloc(sizeof(float) * (BLOCKL_MAX * 3));
       break;
   case -1:  /*低沉聲音*/
       bass = 2;
       d = malloc(sizeof(float) * (BLOCKL_MAX * 2));
       break;
   case 1:   /*正常聲音*/
       d = malloc(sizeof(float) * (BLOCKL_MAX));
       break;
   case 2:   /*高亢聲音*/
       d = malloc(sizeof(float)*(BLOCKL_MAX / 2));
       break;
   case 3:   /*更高亢聲音*/ 
       d = malloc(sizeof(float) * (BLOCKL_MAX / 3));
       break;
}
```

- if放高頻處理的部分
    - 將波長縮短的方式：frequency_index個波一組，並選一個代表存進d裡
- else放低頻處理的部分
    - 將波長拉長的方式：每個波都重複requency_index，並將其全部存進d裡面
- 將d的結果放進data裡，最多BLOCKL_MAX個
```cpp
  if (frequency_index > 0) {
     
   for (int i = 0; i < BLOCKL_MAX; i += frequency_index) {
         d[i / frequency_index] = data[i];
   }
   for (int f = 0; f < frequency_index; f++) {
       for (int i = 0; i < BLOCKL_MAX / frequency_index; i++) {
           data[(BLOCKL_MAX / frequency_index) * f + i] = d[i];
       }
   }
 }/*高音處理*/
 else {
     for (int i = 0; i < BLOCKL_MAX/bass; i ++) {
         for (int c = 0; c < bass; c++) {
           d[c+i*bass] = data[i];
         }
     }
     for (int i = 0; i < BLOCKL_MAX; i++) {
         data[i] = d[i];
     }

 }/*低音處理*/
 free(d);
```
