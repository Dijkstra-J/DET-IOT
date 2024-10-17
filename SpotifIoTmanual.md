# Arduino controled spotify (kinda)
This manual will show you how to set up an arduino program that can control spotify. At first through button, second through bluetooth proximity.

This project is a proof of concept for the concpet described in this document (only in dutch).

## Hardware requirements for the first part
- ESP 32 (or a different board that supports bluetooth and or wifi).
  	- The first part can also be done without bluetooth support.
	- The manual is written using a NodeMCU-ESP32, but it could also be possible with a different board.
<img src="https://github.com/user-attachments/assets/0204c9f2-17ef-44d7-a0f6-832d4507cd95" alt="ESP32" width=400/>


- Button
	- I use a 4 pin button on a 3 pin connector. So if you use a different button some code may have to be changed.
<img src="https://github.com/user-attachments/assets/4a51a36c-0e47-4846-a971-fc0f08795209" alt="Button" width=400/>

## Extra hardware requirements for the second part
- A device that can set up a hotspot (mobile phone works better than pc most of the time).
- A bluetooth device (I will try to set it up with a Garmin watch, but will resort back to mobile phone if that appears troublesome).

## Software requirements
- Arduino IDE

## Library requirements Arduino
- SpotifyArduino https://github.com/witnessmenow/spotify-api-arduino
- ArduinoJson

## API requirements
- Spotify API
	- The options are very limited for a spotify free account, to be able to do anything meaningfull you need a spotify premium subscription. That being said, this manual was written using a spotify free account.

## Part 1, button
For anything to work we need a spotify API, you can create it at https://developer.spotify.com/.  
For starters we make spotify react to a button. All I am interested in for now is just a play/pause toggle, because this is just a proof of concept. To achieve this, I took this project https://github.com/witnessmenow/spotify-api-arduino/blob/main/examples/playerControls/playerControls.ino and modified it to fit my needs. For this to work you need a library you can find here https://github.com/witnessmenow/spotify-api-arduino. To donwload it, select the green "code" button and select "download ZIP". Then in your Arduino IDE go to sketch > include library > add .zip library and add the downloaded file.

Once the Output window says "Library installed" restart your Arduino IDE and open the spotifyarduino example "getRefreshToken". You will need this token later.
In the corresponding places in the file add your spotify developer Client ID & Client Secret. And also add in the corresponding places your hotspot name and password.

The provided code then is a bit unclear but you have to do the following:

1. In the serial monitor find the IP address
2. Go to your spotify API settings and add "http://[IP address (the one from the serial monitor)]/callback/" as a redirect URI (and save your changes)
3. Then on the device that is the hotspot go to a webbrowser and search for "http://[IP address (the one form the Serial monitor)]
4. You should get to a blank page with a spotify authentication link, click the link and authenticate. This should redirect you to the callback page on which the refresh token is shown.

Then go back tot the player controls example and add all the required information (spotify developer Client ID & Client Secret, Hotpsot name and password and spotify refresh token and correct country code (in the market spot)).

Here I ran into a little issue, where it would not be able to send the request. Printing failed to send request on everything.

This appeared to be because of a combination of me being poor (not having spotify premium) and spotify being not a nice company and locking almost all the API stuff behind premium.

So I switchted to a part of the API that doesn't require premium, which is the getCurrentlyPlaying example.

In this example add all the required information (spotify developer Client ID & Client Secret, Hotpsot name and password and spotify refresh token and the correct country code (in the market spot)).

Which did work (correcly even), mark me relieved.

So that makes a change of plans. Instead of pausing and resuming, the code will now request the currently playing song on button pressed. A bit of a shame, but it will serve as the proof of concept. 

If you do have spotify premium, all other functions should be available to you as well. And can (probalby) be implemented in a similar way.

### Start creating own code (finally)
If you have a 3 pin button it should be wired like this:
On the button bourd you should see an S a - and a middle pin. The S pin goes to the data pin (D15 in my case) the - pin goes to the power output (3v3 in my case) and the middle pin goes to the ground. Make sure to test whether this works with a more simple program (the button example with a few added println statements works fine) to prevent getting spammed with requests.

Currently the code requests the currently playing content every minute. I will change this into a request on button press.

I happen to have some code lying around from a previous project that works with buttons, so that saves me a lot of time.

I define a button pin after the Spotify_refersh token
```#define BUTTON_PIN 15``` 
In my case it is pin (D)15

Then at the end of the void setup I add 
``` pinMode(BUTTON_PIN, INPUT_PULLUP);```
Then at the start of the void loop I add ```int buttonState = digitalRead(BUTTON_PIN); ``` and replace the ```if (millis() > requestDueTime)``` with ```if (buttonState == High)``` 

At the end of the void loop I replace the ```requestDueTime = millis() + delayBetweenRequests; ``` with ```delay(150);``` to prevent bounce.

The ```unsigned long delayBetweenRequests = 60000; // Time between requests (1 minute)
unsigned long requestDueTime; //time when request due```
can also be removed, but you can also leave them. So I just comment out them.

Somewhere in the void loop I also added a ```Serial.println(buttonState); ``` for debugging. Do comment out that part once you are done with it, or you will forget and regret ever putting it in.
With that you the first part of the code and the first proof of concept is done. (Full code can be found a the bottom of this file).

### Part 2 Bluetooth connection
The plan was to pause spotify when the bluetooth device got too far away. But since I need premium for that it will now be to pull the currently playing data when the bluetooth device is near.  
To see if I can pull anything from bluetooth, I opened an example about bluetooth bt_classic_device_discovery. I think this will get me the info I need to get a bluetooth connection working.  
This did not get the device I wanted, but did get a few other devices and some data I like to see, which includes something that could be distance.  
The examples didn't get me any further, so I went to google. Which also didn't really help me, but did indicate it is kinda possible, just not too accurate.  
But nothing about functional code still, which led me to chatGPT. [https://chatgpt.com/share/670fa09c-4d0c-800d-9671-41ab1ca80d7e]. Which resulted in the following error: Compilation error: 'init' is not a member of 'BLEDevice'. So a little google searching and screwing around later I figured out it might be a library I don't need causing the issue, so after removing that I got a little furhter.  
But ran into this error Compilation error: conversion from 'BLEScanResults*' to non-scalar type 'BLEScanResults' requested.
It that a "pointer" (whatever that is) is assigned to a non "pointer" object. It appears that the code GPT gave me was incorrect. It included pBLEScan->start(5), where start() is not an existing function in the library. So I changed it into a lot of different variations, but nothing worked.  
That made it time to go reading the library. Which did not at all improve my understanding of the code, so it was time to open the scan example.  
That resulted into me discovering BLEScanResults foundDevices = pBLEScan->start(5); should have been BLEScanResults *foundDevices = pBLEScan->start(5, false);. Mildly annoyed.  
Then there were 2 more places where a . should be replaced by a ->, but luckely the Arduino IDE was able to tell me where that was the case. And then I finally got the code to the board.  

In the exceptionally rare case a fire alarm goes of in your building at around an about exactly this point, while you try to upload the newest version. You may get a Failed uploading: uploading error: exit status 1, when you pack your stuff.

The code started printing like this:
14:44:09.931 -> _string: construction from null is not valid
14:44:18.098 -> ring: construction from null is not valid
14:44:26.247 -> basic_string: construction from null is not valid
14:44:34.399 -> <Unknown>
14:44:42.574 -> 
14:44:50.741 ->  construction from null is not valid
14:44:58.861 -> ng: construction from null is not valid
Which I assumed to mean there were no results.

Adding some println statemenst I discovered that the functions used to call the device information were always empty.
Adding some delays behind those statements gave me the RSSI's (used for getting the distance), but not the data I need to get it from a specific device.  
Very sometimes a name popped up, but mostly it was empty even with a bigger delay.  
So I switched to trying to get the addresses from the devices.  
When I finally got that to work I noticed all the adresses used lower case letters and not upper case letters, as I had seen when I read the mac-address from my phone. So I changed the case of the letters in my mac-adress to search.  

Then I tried to turn off the random mac address, that increases privacy, but might find it harder for the arduino to find the device. And also went to the bluetooth settings and made the computer I am using to achieve this discoverable. This also did not work, but I tried to do it with one of the devices I found in the list. I chose the one with the highest RSSI (which I assumed would be the closest). This way I could at least check whether the rest of the code was functional. This worked but, the returned distance was around 23 meters which means it was probably the most far away object. With this knowledge now I checked the address with the lowest RSSI value. This time the value returned was 8 meters. Which confirmed my new idea that lower RSSI values mean lower distances.

I have no idea why I can't find the device I want to find and also can't find out why I very rarely get device names, although this might have something to do with privacy and security.  
So to be able to create my final system, I will make the trigger do the following: Everytime a Bluetooth device gets within an RSSI of -71 (around 4 meters) get the currently playing song from the spotify API (Full code can be found a the bottom of this file). Do keep in mind that RSSI is by no means a very accurate reprsentation of distance it can easily fluctuate with multiple meters.

### Part 3 Combining part 1 and 2
For the final part I take the first two parts and combine them into one piece of code.
The code is mainly build up from the code of part one with the essential parts of part 2. In the loop a second if statement is added that looks like this:

``` if (device.getRSSI() < 75){ ```

Whith in that statment the existing code for finding the currently playing song.
While copying code do not forget this part ``` BLEScan* pBLEScan; ```. It should go after the libary calls and before the void setup.  

The code for the button press also still remains to keep the system operational without bluetooth.
It is possible that the code gets stuck in the compiling phase, just give it time, the code is either quite big or very poorly optimised.  
The previous line immediately came back to bite me. The program was too long and didn't fit on my board. Which meant I had to reduce the length by about 40%.  
The simplest solution I tried was to remove a load of comments. Which not too surprisingly did nothing at all.
So I removed all the code required for the button. I didn't really want to do this, but I had to to make it work.
This removed only 1 percent of the overshot length. So I removed some libraries I simply hoped I did not need.
This was still not enough, so I went to chatGPT to shorten it even further. Which also achieved nothing at all.

### Code for part 1
```C
/*******************************************************************
    Prints your currently playing track on spotify to the
    serial monitor using an ES32 or ESP8266

    NOTE: You need to get a Refresh token to use this example
    Use the getRefreshToken example to get it.

    Compatible Boards:
	  - Any ESP8266 or ESP32 board

    Parts:
    ESP32 D1 Mini style Dev board* - http://s.click.aliexpress.com/e/C6ds4my

 *  * = Affiliate

    If you find what I do useful and would like to support me,
    please consider becoming a sponsor on Github
    https://github.com/sponsors/witnessmenow/


    Written by Brian Lough
    YouTube: https://www.youtube.com/brianlough
    Tindie: https://www.tindie.com/stores/brianlough/
    Twitter: https://twitter.com/witnessmenow
 *******************************************************************/

// ----------------------------
// Standard Libraries
// ----------------------------

#if defined(ESP8266)
#include <ESP8266WiFi.h>
#elif defined(ESP32)
#include <WiFi.h>
#endif

#include <WiFiClientSecure.h>

// ----------------------------
// Additional Libraries - each one of these will need to be installed.
// ----------------------------

#include <SpotifyArduino.h>
// Library for connecting to the Spotify API

// Install from Github
// https://github.com/witnessmenow/spotify-api-arduino

// including a "spotify_server_cert" variable
// header is included as part of the SpotifyArduino libary
#include <SpotifyArduinoCert.h>

#include <ArduinoJson.h>
// Library used for parsing Json from the API responses

// Search for "Arduino Json" in the Arduino Library manager
// https://github.com/bblanchon/ArduinoJson

//------- Replace the following! ------

char ssid[] = "your hotspot name here";         // your network SSID (name)
char password[] = "your wifi password here"; // your network password

char clientId[] = "your client id here";     // Your client ID of your spotify APP
char clientSecret[] = "Your client secret here"; // Your client Secret of your spotify APP (Do Not share this!)

// Country code, including this is advisable
#define SPOTIFY_MARKET "NL"

#define SPOTIFY_REFRESH_TOKEN "Your refersh token here"

#define BUTTON_PIN 15

//------- ---------------------- ------

WiFiClientSecure client;
SpotifyArduino spotify(client, clientId, clientSecret, SPOTIFY_REFRESH_TOKEN);

//unsigned long delayBetweenRequests = 60000; // Time between requests (1 minute)
//unsigned long requestDueTime;               //time when request due

void setup()
{

    Serial.begin(115200);

    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, password);
    Serial.println("");

    // Wait for connection
    while (WiFi.status() != WL_CONNECTED)
    {
        delay(500);
        Serial.print(".");
    }
    Serial.println("");
    Serial.print("Connected to ");
    Serial.println(ssid);
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());

    // Handle HTTPS Verification
#if defined(ESP8266)
    client.setFingerprint(SPOTIFY_FINGERPRINT); // These expire every few months
#elif defined(ESP32)
    client.setCACert(spotify_server_cert);
#endif
    // ... or don't!
    //client.setInsecure();

    // If you want to enable some extra debugging
    // uncomment the "#define SPOTIFY_DEBUG" in SpotifyArduino.h

    Serial.println("Refreshing Access Tokens");
    if (!spotify.refreshAccessToken())
    {
        Serial.println("Failed to get access tokens");
    }
  pinMode(BUTTON_PIN, INPUT_PULLUP);
}

void printCurrentlyPlayingToSerial(CurrentlyPlaying currentlyPlaying)
{
    // Use the details in this method or if you want to store them
    // make sure you copy them (using something like strncpy)
    // const char* artist =

    Serial.println("--------- Currently Playing ---------");

    Serial.print("Is Playing: ");
    if (currentlyPlaying.isPlaying)
    {
        Serial.println("Yes");
    }
    else
    {
        Serial.println("No");
    }

    Serial.print("Track: ");
    Serial.println(currentlyPlaying.trackName);
    Serial.print("Track URI: ");
    Serial.println(currentlyPlaying.trackUri);
    Serial.println();

    Serial.println("Artists: ");
    for (int i = 0; i < currentlyPlaying.numArtists; i++)
    {
        Serial.print("Name: ");
        Serial.println(currentlyPlaying.artists[i].artistName);
        Serial.print("Artist URI: ");
        Serial.println(currentlyPlaying.artists[i].artistUri);
        Serial.println();
    }

    Serial.print("Album: ");
    Serial.println(currentlyPlaying.albumName);
    Serial.print("Album URI: ");
    Serial.println(currentlyPlaying.albumUri);
    Serial.println();

    if (currentlyPlaying.contextUri != NULL)
    {
        Serial.print("Context URI: ");
        Serial.println(currentlyPlaying.contextUri);
        Serial.println();
    }

    long progress = currentlyPlaying.progressMs; // duration passed in the song
    long duration = currentlyPlaying.durationMs; // Length of Song
    Serial.print("Elapsed time of song (ms): ");
    Serial.print(progress);
    Serial.print(" of ");
    Serial.println(duration);
    Serial.println();

    float percentage = ((float)progress / (float)duration) * 100;
    int clampedPercentage = (int)percentage;
    Serial.print("<");
    for (int j = 0; j < 50; j++)
    {
        if (clampedPercentage >= (j * 2))
        {
            Serial.print("=");
        }
        else
        {
            Serial.print("-");
        }
    }
    Serial.println(">");
    Serial.println();

    // will be in order of widest to narrowest
    // currentlyPlaying.numImages is the number of images that
    // are stored
    for (int i = 0; i < currentlyPlaying.numImages; i++)
    {
        Serial.println("------------------------");
        Serial.print("Album Image: ");
        Serial.println(currentlyPlaying.albumImages[i].url);
        Serial.print("Dimensions: ");
        Serial.print(currentlyPlaying.albumImages[i].width);
        Serial.print(" x ");
        Serial.print(currentlyPlaying.albumImages[i].height);
        Serial.println();
    }
    Serial.println("------------------------");
}

void loop()
{
  int buttonState = digitalRead(BUTTON_PIN);
  //Serial.println(buttonState);
    if (buttonState == HIGH)
    {
        Serial.print("Free Heap: ");
        Serial.println(ESP.getFreeHeap());

        Serial.println("getting currently playing song:");
        // Market can be excluded if you want e.g. spotify.getCurrentlyPlaying()
        int status = spotify.getCurrentlyPlaying(printCurrentlyPlayingToSerial, SPOTIFY_MARKET);
        if (status == 200)
        {
            Serial.println("Successfully got currently playing");
        }
        else if (status == 204)
        {
            Serial.println("Doesn't seem to be anything playing");
        }
        else
        {
            Serial.print("Error: ");
            Serial.println(status);
        }
        delay(150);
    }
}
```

### Code for part 2
```C
#include <BLEDevice.h>
#include <BLEScan.h>
#include <BLEAdvertisedDevice.h>

BLEScan* pBLEScan;

void setup() {
  Serial.begin(115200);
  while(!Serial);
  BLEDevice::init("");  // Initialize BLE
  pBLEScan = BLEDevice::getScan();  // Create a BLE scanner object
  pBLEScan->setActiveScan(true);    // Set scanning mode to active (faster)
}

void loop() {
  BLEScanResults *foundDevices = pBLEScan->start(10, false);  // Scan for 5 seconds
  int deviceCount = foundDevices->getCount();         // Get the number of found devices
  Serial.print("Scan done, devices found: ");
  Serial.println(deviceCount);
  delay(500);
  Serial.println("Start searching for specified device");
  for (int i = 0; i < deviceCount; i++) {
    BLEAdvertisedDevice device = foundDevices->getDevice(i);
    Serial.println(device.getAddress().toString());
    Serial.println(device.getName());
    Serial.println(device.getRSSI());

    // Check if this is the target Bluetooth device (match by address or name)
    if (device.getAddress().toString() == "XX:XX:XX:XX:XX:XX") { // Replace with your target device's address
      int rssi = device.getRSSI(); // Get RSSI value
      Serial.print("Device found: ");
      Serial.println(device.getAddress().toString().c_str());
      Serial.print("RSSI: ");
      Serial.println(rssi);

      // Optionally, calculate approximate distance (based on RSSI)
      float distance = calculateDistance(rssi);
      Serial.print("Estimated Distance: ");
      Serial.print(distance);
      Serial.println(" meters");
    }
  }
  pBLEScan->clearResults();  // Clear scan results
  delay(3000);  // Wait before the next scan
}

float calculateDistance(int rssi) {
  int txPower = -59;  // Typical RSSI value at 1 meter distance
  if (rssi == 0) {
    return -1.0; // if we cannot determine distance (signal strength is 0)
  }

  float ratio = rssi * 1.0 / txPower;
  if (ratio < 1.0) {
    return pow(ratio, 10);
  } else {
    float distance = (0.89976) * pow(ratio, 7.7095) + 0.111;
    return distance;
  }
}
```

### Code for part 3 (Final code)
```C

```
