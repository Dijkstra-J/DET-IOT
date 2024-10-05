# Guide for using Seeed AI vision


## Required hardware
- NodeMCU (or a different board with that supports wifi)
  - If u use a different board, it is possible you have to change some of the code and need to install different libaries)
![nodemcu](https://github.com/user-attachments/assets/3315e651-f0f5-4989-92cf-d5a550946391)
- SeeeD AI vision module
![camera](https://github.com/user-attachments/assets/cd24d0b6-15d2-4bf3-90b8-03196a5f8a74)

## Required libraries
- Go to: https://github.com/Seeed-Studio/Seeed_Arduino_GroveAI
-   From releases download the Source code zip file
-   In your Arduino IDE, open the sketch menu and select include library, add .ZIP library
-   Once your output winows reports "Library installed" the library is ready to use.
-   (You may have to restart your IDE for the library to be usable)

## Other requirements
- Windows computer with a browser

## Failed (for now)
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

