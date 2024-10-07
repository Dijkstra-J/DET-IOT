# Guide for using Seeed AI vision


## Required hardware
- NodeMCU (or a different board with that supports wifi)
  - If u use a different board, it is possible you have to change some of the code and need to install different libaries)

![nodemcu](https://github.com/user-attachments/assets/3315e651-f0f5-4989-92cf-d5a550946391)
- SeeeD AI vision module

![camera](https://github.com/user-attachments/assets/cd24d0b6-15d2-4bf3-90b8-03196a5f8a74)

## Required libraries
- Go to: https://github.com/Seeed-Studio/Seeed_Arduino_GroveAI
- On this page click the code button and select download ZIP
-   In your Arduino IDE, open the sketch menu and select include library, add .ZIP library
-   Once your output winows reports "Library installed" the library is ready to use.
-   (You may have to restart your IDE for the library to be usable)

## Other requirements
- Windows computer with a browser
- Arduino IDE

## Preperation
- To setup get the vision module and attach it to your laptop. This should prompt a notification from your pc that Grove AI is detected click this message. It should lead you to this link: https://files.seeedstudio.com/grove_ai_vision/index.html
- Then connect your Vision module to your MCU, the black wire should go to GND, the red wire to 3V3, the yellow wire to D1 and the white wire to D2 (if the code prints "Algo begin failed" in the serial monitor try switching the yellow and white wires.
- Connect your MCU to your computer

##Code
Next is the code, it is basically the example you can find in your Arduino IDE File > Examples > Seeed_Arduino_GroveAI. With some small additions
```C
#include "Seeed_Arduino_GroveAI.h"
#include <Wire.h>

GroveAI ai(Wire);
uint8_t state = 0;
void setup()
{
  Wire.begin();
  Serial.begin(115200);
  while(!Serial); // I added this and the next line, to make sure this is printed to the serial monitor. It would otherwise have passed this step before my Serial monitor was started up not printing this and not showing me the problems.
  delay(10000);
   Serial.println("begin");
  if (ai.begin(ALGO_OBJECT_DETECTION, (MODEL_INDEX_T)0x11)) // Object detection and pre-trained model 1 MODEL_PRE_INDEX_1
  {
    Serial.print("Version: 0x");
    Serial.println(ai.version(), HEX);
    Serial.print("ID: 0x");
    Serial.println( ai.id(), HEX);
    Serial.print("Algo: ");
    Serial.println( ai.algo());
    Serial.print("Model: ");
    Serial.println(ai.model());
    Serial.print("Confidence: ");
    Serial.println(ai.confidence());
    state = 1;
  }
  else
  {
    Serial.println("Algo begin failed.");
  }
}

void loop()
{
  if (state == 1)
  {
    uint32_t tick = millis();
    if (ai.invoke()) // begin invoke
    {
      while (1) // wait for invoking finished
      {
        CMD_STATE_T ret = ai.state(); 
        if (ret == CMD_STATE_IDLE)
        {
          break;
        }
        delay(20);
      }

     uint8_t len = ai.get_result_len(); // receive how many people detect
     if(len)
     {
       int time1 = millis() - tick; 
       Serial.print("Time consuming: ");
       Serial.println(time1);
       Serial.print("Number of people: ");
       Serial.println(len);
       object_detection_t data;       //get data

       for (int i = 0; i < len; i++)
       {
          Serial.println("result:detected");
          Serial.print("Detecting and calculating: ");
          Serial.println(i+1);
          ai.get_result(i, (uint8_t*)&data, sizeof(object_detection_t)); //get result
  
          Serial.print("confidence:");
          Serial.print(data.confidence);
          Serial.println();
        }
     }
     else
     {
       Serial.println("No identification");
     }
    }
    else
    {
      delay(1000);
      Serial.println("Invoke Failed.");
    }
  }
  else
  {
    state == 0;
  }
}

```

## Earlier failure because I didn't donwload the library correctly...
After connecting and setting everything up, uploading the file did...
Well, nothing.

![Seeed](https://github.com/user-attachments/assets/b9ab8fb2-c545-45d3-b29a-39b929bb2193)

The seeed web app showed this in all my browsers and the serial monitor printed nothing. I found out that could be helped by adding some delay, but that only resulting in printing there was an error.

At least now I knew. Searching the internet for said error indicated that you had to check the model number.

No big deal, so I discoverd my vison module did not have a model installed at all.

So, I donwloaded the model and followed the instructions on how to install it. Sadly (possibly a windows 11 problem), everytime I tried to install (which was many times, trust me) the module disconnected before the install could be completed.

So then there were some more suggestions including resetting the dvice to factorysettings and all, but since it is not mine, I did not really want to do that. And I had to download some shady software as well, which I also didn't really feel like doing.

And I searched the internet for stuff as indicated in this image:

![tabs](https://github.com/user-attachments/assets/11bafb38-546b-4aac-9cb3-37b48e5b8400)

