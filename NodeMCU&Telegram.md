# Guide for managing lights on the NodeMCU using telegram
This guide shows you how to control lights attached to a NodeMCU through Telegram.

## Required hardware
- NodeMCU (or a different board with that supports wifi)
  - If u use a different board, it is possible you have to change some of the code and need to install different libaries)
    ![nodemcu](https://github.com/user-attachments/assets/3315e651-f0f5-4989-92cf-d5a550946391)
- Adafruit neopixel LED strip
  - Or a different type of LED, in which case you will once again need to change some code
    ![LED_strip](https://github.com/user-attachments/assets/57e94898-f768-4494-88cc-8394bb757e9f)
- A smartphone that can set up a mobile hotspot

## Required libraries
- Adafruit NeoPixel (and dependancies)
- ArduinoJson (and dependancies)
- AsyncTelegram2 (and dependancies)
To donwload libraries, in your Arduino IDE, go to Tools > Manage libraries and search for the required libraries

![Libraries](https://github.com/user-attachments/assets/7ea6ba26-a6e9-4298-9d6c-6a64bcb963ed)

## Other requirements
- Telegram account (can be created in the Telegram mobile app)

## Creating the telegram bot
1. Open the Telegram app
2. Search for the account @BotFather

![BotFather](https://github.com/user-attachments/assets/3096c19e-12bb-485b-8a38-8afeb0c9a6bb)

3. Start the bot and send and send "/newbot"
4. Give the bot a name (the name must be unique, so it may take a few tries)

## Code
Nextup is the code required to make it all work. It is basicaly the echobot by Tolentino Cotesta with a few additions.
```C
/*Based on:
  Name:        echoBot.ino
  Created:     26/03/2021
  Author:      Tolentino Cotesta <cotestatnt@yahoo.com>
  Description: a simple example that check for incoming messages
               and reply the sender with the received message.
                 The message will be forwarded also in a public channel
                 anad to a specific userid.
*/


/* 
  Set true if you want use external library for SSL connection instead ESP32@WiFiClientSecure 
  For example https://github.com/OPEnSLab-OSU/SSLClient/ is very efficient BearSSL library.
  You can use AsyncTelegram2 even with other MCUs or transport layer (ex. Ethernet)
  With SSLClient, be sure "certificates.h" file is present in sketch folder
*/ 
#define USE_CLIENTSSL true  //I have no idea what this does, so I didn't touch it.
```
First we will call the required libraries and define some values. 
```C
#include <AsyncTelegram2.h>
#include <Adafruit_NeoPixel.h>

#define LED_PIN    D1          // The pin on your board that is connected to the dataport on your LED strip
#define NUM_LEDS 10            // Number of leds on your LED strip

Adafruit_NeoPixel strip(NUM_LEDS, LED_PIN, NEO_GRB + NEO_KHZ800); // The code that allows you to use all the strip.XX code later on.
```

Next up is the code that prepares for the wifi setup. I did not create it so I don't really understand it, but it works.
```C
// Timezone definition
#include <time.h>
#define MYTZ "CET-1CEST,M3.5.0,M10.5.0/3"

#ifdef ESP8266
  #include <ESP8266WiFi.h>
  BearSSL::WiFiClientSecure client;
  BearSSL::Session   session;
  BearSSL::X509List  certificate(telegram_cert);
  
#elif defined(ESP32)
  #include <WiFi.h>
  #include <WiFiClient.h>
  #if USE_CLIENTSSL
    #include <SSLClient.h>  
    #include "tg_certificate.h"
    WiFiClient base_client;
    SSLClient client(base_client, TAs, (size_t)TAs_NUM, A0, 1, SSLClient::SSL_ERROR);
  #else
    #include <WiFiClientSecure.h>
    WiFiClientSecure client;  
  #endif
#endif

AsyncTelegram2 myBot(client);
```
In the part below you add the details of your hotspot and the token of your telegram bot. You can also add a userID to your bot, to prevent it from being used by others.
```C
const char* ssid  =  "*Insert your hotspot name here*";     // SSID WiFi network
const char* pass  =  "*Insert your hotspot password here*";     // Password  WiFi network
const char* token =  "*Insert the telegram token of your bot here*";  // Telegram token

// Target user can find it's own userid with the bot @JsonDumpBot
// https://t.me/JsonDumpBot
int64_t userid = 123456789;  


// Name of public channel (your bot must be in admin group) (Once again no clue what this does, I don't think it is necesary, but I left it just in case)
const char* channel = "@tolentino_cotesta";
```
The next part is to set up the rainbow lights, so you can call them later.
```C
uint32_t Wheel(byte WheelPos) {
  WheelPos = 255 - WheelPos;
  if (WheelPos < 85) {
    return strip.Color(255 - WheelPos * 3, 0, WheelPos * 3);
  } else if (WheelPos < 170) {
    WheelPos -= 85;
    return strip.Color(0, WheelPos * 3, 255 - WheelPos * 3);
  } else {
    WheelPos -= 170;
    return strip.Color(WheelPos * 3, 255 - WheelPos * 3, 0);
  }
}

// Function for the rainbow cycle effect
void rainbowCycle(uint8_t wait) {
  for (int j = 0; j < 256 * 5; j++) { // 5 cycles of all colors on the wheel
    for (int i = 0; i < strip.numPixels(); i++) {
      strip.setPixelColor(i, Wheel((i * 256 / strip.numPixels() + j) & 255));
    }
    strip.show();
    delay(wait);
  }
}
```
Then the setup, you need to do a few things here:
- Send a begin command to the LED strip
- Begin the serial monitor (you can open it with Ctrl + Shift + m)
-   Notice that you have to set the serial monitor baut rate the same as the baut rate you give it in the Serial.begin command

![Serial Monitor](https://github.com/user-attachments/assets/5a0904fe-cc84-420a-8796-dda4d4739391)

- Then the Wifi mode is declared, which will print a number of "." until it is connected.
- Then the time is synced, no idea how or why that works, but it seems important.
- Then the telegram bot is setup, with a refresh time of 2 seconds
- Then there is a Serial.println that writes whether the bot is okay, but I have noticed that the bot works when it is NOK (not okay), so the best way to test whether it is operational is to just send a message and see if you get a response

```C
void setup() {
  strip.begin();
  // initialize the Serial
  Serial.begin(115200);
  Serial.println("\nStarting TelegramBot...");

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, pass);
  delay(500);
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print('.');
    delay(500);
  }
 
#ifdef ESP8266
  // Sync time with NTP, to check properly Telegram certificate
  configTime(MYTZ, "time.google.com", "time.windows.com", "pool.ntp.org");
  //Set certficate, session and some other base client properies
  client.setSession(&session);
  client.setTrustAnchors(&certificate);
  client.setBufferSizes(1024, 1024);
#elif defined(ESP32)
  // Sync time with NTP
  configTzTime(MYTZ, "time.google.com", "time.windows.com", "pool.ntp.org");
  #if USE_CLIENTSSL == false
    client.setCACert(telegram_cert);
  #endif
#endif
  
  // Set the Telegram bot properies
  myBot.setUpdateTime(2000);
  myBot.setTelegramToken(token);

  // Check if all things are ok
  Serial.print("\nTest Telegram connection... ");
  myBot.begin() ? Serial.println("OK") : Serial.println("NOK");

  char welcome_msg[128];
  snprintf(welcome_msg, 128, "BOT @%s online\n/help all commands avalaible.", myBot.getBotName());

  // Send a message to specific user who has started your bot
  myBot.sendTo(userid, welcome_msg);
}
```
Lastly the void loop, that loops all the code that is necesary.
- The first part is simply the echobot
- Then for the modes red, green, blue, white, off and rainbow an if statement is written that works like this:
-   If message contains "a color" set all the leds in the strip to that color.

```C
void loop() {
  // local variable to store telegram message data
  TBMessage msg;

  // if there is an incoming message...
  if (myBot.getNewMessage(msg)) {    
    // Send a message to your public channel
    String message ;
    message += "Message from @";
    message += myBot.getBotName();
    message += ":\n";
    message += msg.text;
    Serial.println(message);
    myBot.sendToChannel(channel, message, true);
    // echo the received message
    myBot.sendMessage(msg, msg.text);
    Serial.println("Reply send");  

    if (message.indexOf("blue")>0) {
      Serial.println("Input blue");
      for (int i=0; i < NUM_LEDS; i++){
        strip.setPixelColor(i, strip.Color(0, 0, 255));
      }
      strip.show();     
    } else if (message.indexOf("red")>0) {
      Serial.println("Input red");
      for (int i=0; i < NUM_LEDS; i++){
        strip.setPixelColor(i, strip.Color(255, 0, 0));
      }
      strip.show();
    } else if (message.indexOf("green")>0) {
      Serial.println("Input green");
      for (int i=0; i < NUM_LEDS; i++){
        strip.setPixelColor(i, strip.Color(0, 255, 0));
      }
      strip.show();
    } else if (message.indexOf("on")>0) {
      Serial.println("Input on");
      for (int i=0; i < NUM_LEDS; i++){
        strip.setPixelColor(i, strip.Color(255, 255, 255));
      }
      strip.show();
    } else if (message.indexOf("off")>0) {
      Serial.println("Input off");
      strip.clear();
      strip.show();
    } else if (message.indexOf("rainbow")>0) {
      Serial.println("Input rainbow");
      rainbowCycle(1); // Adjust speed by changing the delay value
    } else {
      Serial.println(message);
      Serial.println("No color was mentioned");
    }
  }
}
```

With that all the code is ready and your bot is ready to be used.

## Common issues:

## Sources:
https://icthva.sharepoint.com/:w:/s/FDMCI_ORG__CMD-Amsterdam/Eb7Jd27yWphMuVFbMHV_9WoBEg5_zqAQilsb6Q3gPSKueg?e=f5PM7l

https://www.arduino.cc/reference/en/language/variables/data-types/string/functions/equalsignorecase/

https://forum.arduino.cc/t/test-if-a-string-contains-a-string/478927/3 

https://chatgpt.com/share/66fe628d-03c0-800d-86ae-27fbf8e74537
