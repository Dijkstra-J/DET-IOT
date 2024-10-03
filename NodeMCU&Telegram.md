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
#define USE_CLIENTSSL true  

#include <AsyncTelegram2.h>
#include <Adafruit_NeoPixel.h>
#define LED_PIN    D1          // LED strip data pin connected to D5 (GPIO14)
#define NUM_LEDS 10

Adafruit_NeoPixel strip(NUM_LEDS, LED_PIN, NEO_GRB + NEO_KHZ800);

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

const char* ssid  =  "*Insert your hotspot name here*";     // SSID WiFi network
const char* pass  =  "*Insert your hotspot password here*";     // Password  WiFi network
const char* token =  "*Insert the telegram token of your bot here*";  // Telegram token

// Target user can find it's own userid with the bot @JsonDumpBot
// https://t.me/JsonDumpBot
int64_t userid = 123456789;  


// Name of public channel (your bot must be in admin group)
const char* channel = "@tolentino_cotesta";

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

void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
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

void loop() {
  
  static uint32_t ledTime = millis();
  if (millis() - ledTime > 150) {
    ledTime = millis();
    digitalWrite(LED_BUILTIN, !digitalRead(LED_BUILTIN));
  }

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
