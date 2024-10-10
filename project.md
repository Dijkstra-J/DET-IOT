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
-   The first part can also be done with a NodeMCU.
-   The manual is written using an ESP 32, but it could also be possible with a different board.

## Extra hardware requirements for the second part
- A device that can set up a hotspot (mobile phone works better than pc most of the time).
- A bluetooth device (I will try to set it up with a Garmin watch, but will resort back to mobile phone if that appears troublesome).

## Software requirements
- Arduino IDE
- A Python IDE
-   I use Visual Studio Code in an Anaconda environment

## Library requirements Arduino
- Uhm, still figuring that out

## Library requirements Python
- SwSpotify https://pypi.org/project/swspotify/
- pySerial https://pyserial.readthedocs.io/en/latest/index.html
- PyAutoGui https://www.hackster.io/akshithk/arduino-controlled-spotify-7fa4b0

## Extensions
- Arduino extension for Visual Studio code https://github.com/vscode-arduino/vscode-arduino

## API requirements
- Spotify API
