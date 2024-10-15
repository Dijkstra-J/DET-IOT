Since I need to make something connected to the internet

I will see if I can get the arduino to control my spotify. For starters I wil see if I can make it pause and play with a button.

It would also be funny if I could visuallise the audio on an LED strip, not really related, just funny

If I get far enough, I will try to make it play and pause with a proximity sensor (if possible)


Sources that make me think some of this is even remotely possible:

https://www.hackster.io/akshithk/arduino-controlled-spotify-7fa4b0

https://github.com/witnessmenow/spotify-api-arduino

https://projecthub.arduino.cc/shajeeb/32-band-audio-spectrum-visualizer-analyzer-924af5

https://docs.arduino.cc/tutorials/nano-33-ble-sense-rev2/proximity-sensor/

https://forum.arduino.cc/t/esp32-bluetooth-proximity-sensor/1025299

Actual manual starts here:


# Arduino controled spotify
This manual will show you how to set up an arduino program that can control spotify. At first through button, second through bluetooth proximity.

This project is a proof of concept for the concpet described in this document (only in dutch).

## Hardware requirements for the first part
- ESP 32 (or a different board that supports bluetooth and or wifi).
-   The first part can also be done without bluetooth support.
-   The manual is written using a NodeMCU-ESP32, but it could also be possible with a different board.

## Extra hardware requirements for the second part
- A device that can set up a hotspot (mobile phone works better than pc most of the time).
- A bluetooth device (I will try to set it up with a Garmin watch, but will resort back to mobile phone if that appears troublesome).

## Software requirements
- Arduino IDE
- A Python IDE
-   I use Visual Studio Code in an Anaconda environment

## Library requirements Arduino
- SpotifyArduino https://github.com/witnessmenow/spotify-api-arduino
- ArduinoJson

## Library requirements Python
- SwSpotify https://pypi.org/project/swspotify/
- pySerial https://pyserial.readthedocs.io/en/latest/index.html
- PyAutoGui https://www.hackster.io/akshithk/arduino-controlled-spotify-7fa4b0

## Extensions
- Arduino extension for Visual Studio code https://github.com/vscode-arduino/vscode-arduino

## API requirements
- Spotify API

## Part 1, button
For anything to work we need a spotify API, you can create it at https://developer.spotify.com/
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
On the button bourd you should see an S a - and a middle pin. The S pin goes to the data pin (D15 in my case) the - pin goes to the power output (3v3 in my case) and the middle pin goes to the ground.
Currently the code requests the currently playing content every minute. I will change this into a request on button press.
I happen to have some code lying around from a previous project that works with buttons, so that saves me a lot of time.
I define a button pin after the Spotify_refersh token 
```C #define BUTTON_PIN 15``` In my case it is pin (D)15
Then at the end of the void setup I add
```C pinMode(BUTTON_PIN, INPUT_PULLUP);```
Then at the start of the void loop I add ```C int buttonState = digitalRead(BUTTON_PIN); ``` and replace the ```C if (millis() > requestDueTime)``` with ```C if (buttonState == LOW)``` (Not LOW, I did that first and then it jsut constantly goes of).
At the end of the void loop I replace the ```C requestDueTime = millis() + delayBetweenRequests; ``` with ```C delay(150);``` to prevent bounce.
The ```C unsigned long delayBetweenRequests = 60000; // Time between requests (1 minute)
unsigned long requestDueTime;               //time when request due``` can also be removed, but you can also leave them. So I just comment out them.
Somewhere in the void loop I also added a ```C Serial.println(buttonState); ``` for debugging. Do comment out that part once you are done with it, or you will forget and regret ever putting it in.
With that you the first part of the code and the first proof of concept is done. (Full code can be found a the bottom of this file.

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

char ssid[] = "J.D.";         // your network SSID (name)
char password[] = "planetofthekiwis"; // your network password

char clientId[] = "67e0980fa03941609c24ae7126b30ae5";     // Your client ID of your spotify APP
char clientSecret[] = "ff9d5521a0974487a99cddad31f6f8f5"; // Your client Secret of your spotify APP (Do Not share this!)

// Country code, including this is advisable
#define SPOTIFY_MARKET "NL"

#define SPOTIFY_REFRESH_TOKEN "AQAp1IMnhLdwbRwIV_f2i5XTqG8oyatSUPESyZtdYwv21k2AGSwlXH4HaieJTFYgTp3ObhlKxXHWYlJ5XOTlE5oGvYFv9mJWEEOVfjitaA4ZnPIyvjQJ2ltr6RBcziV3988"

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
