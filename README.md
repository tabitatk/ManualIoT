# ManualIoT

- Tabita Krommenhoek
- 26-10-2022
- (500833831)
- IoT

# Requirments
- LED light
- Hardware
- Arduino IDE
- NTPClient library
- Hotspot


# 1. Installing NTP Library

We’ll use the NTPClient library to get time in our arduino so we can connect the time to light. In Arduino IDE we will go to go Manage Libraries. The Library Manager should open. And were going to select the NTP Library.

<img src="/images-manual/NTPCLientLibrary.png" width="200px">


Then open File > Examples > NTP Client > IsTimeSet
After that replace The wifi name and password with your own.
As you see the wifi if not connecting to my board

<img src="/images-manual/wififout.png" width="1000px">

So i got an error, after that i googled for for an Adafruit code with time that works with NTP CLient.
I found this raw code in another manual and changed the wifi name and password to my own and it connected this time.
This is the code:

~~~/*
  Rui Santos
  Complete project details at https://RandomNerdTutorials.com/esp8266-nodemcu-date-time-ntp-client-server-arduino/
  
  Permission is hereby granted, free of charge, to any person obtaining a copy
  of this software and associated documentation files.
  
  The above copyright notice and this permission notice shall be included in all
  copies or substantial portions of the Software.
*/

#include <ESP8266WiFi.h>
#include <NTPClient.h>
#include <WiFiUdp.h>

// Replace with your network credentials
const char *ssid     = "REPLACE_WITH_YOUR_SSID";
const char *password = "REPLACE_WITH_YOUR_PASSWORD";

// Define NTP Client to get time
WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "pool.ntp.org");

//Week Days
String weekDays[7]={"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"};

//Month names
String months[12]={"January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"}

void setup() {
  // Initialize Serial Monitor
  Serial.begin(115200);
  
  // Connect to Wi-Fi
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

// Initialize a NTPClient to get time
  timeClient.begin();
  // Set offset time in seconds to adjust for your timezone, for example:
  // GMT +1 = 3600
  // GMT +8 = 28800
  // GMT -1 = -3600
  // GMT 0 = 0
  timeClient.setTimeOffset(0);
}

void loop() {
  timeClient.update();

  time_t epochTime = timeClient.getEpochTime();
  Serial.print("Epoch Time: ");
  Serial.println(epochTime);
  
  String formattedTime = timeClient.getFormattedTime();
  Serial.print("Formatted Time: ");
  Serial.println(formattedTime);  

  int currentHour = timeClient.getHours();
  Serial.print("Hour: ");
  Serial.println(currentHour);  

  int currentMinute = timeClient.getMinutes();
  Serial.print("Minutes: ");
  Serial.println(currentMinute); 
   
  int currentSecond = timeClient.getSeconds();
  Serial.print("Seconds: ");
  Serial.println(currentSecond);  

  String weekDay = weekDays[timeClient.getDay()];
  Serial.print("Week Day: ");
  Serial.println(weekDay);    

  //Get a time structure
  struct tm *ptm = gmtime ((time_t *)&epochTime); 

  int monthDay = ptm->tm_mday;
  Serial.print("Month day: ");
  Serial.println(monthDay);

  int currentMonth = ptm->tm_mon+1;
  Serial.print("Month: ");
  Serial.println(currentMonth);

  String currentMonthName = months[currentMonth-1];
  Serial.print("Month name: ");
  Serial.println(currentMonthName);

  int currentYear = ptm->tm_year+1900;
  Serial.print("Year: ");
  Serial.println(currentYear);

  //Print complete date:
  String currentDate = String(currentYear) + "-" + String(currentMonth) + "-" + String(monthDay);
  Serial.print("Current date: ");
  Serial.println(currentDate);

  Serial.println("");

  delay(2000);
}
~~~
My code in arduino is now reading the time. 

<img src="/images-manual/correct-time.png" width="300px">

# 2. Adjust timezone

As you can see the serial monitos says its 11:32:44 but on my clock it's 13:32:44 so we have to adjust the timezone were in
We are in the timezone GMT + 2 so that means we have to fill in 7200 in stead of 0.

<img src="/images-manual/time7200.png" width="300px">

# Connecting time to if statement

After you've changed the timezone you can try to make an if-else statement first, to see if it works. 
I googled for an if and else statement in arduino and found this raw code, now you can try to put the code in and see what it does:

const int analogPin = A0;   // pin that the sensor is attached to
const int ledPin = 13;      // pin that the LED is attached to
const int threshold = 400;  // an arbitrary threshold level that's in the range of the analog input

~~~ void setup() {
  // initialize the LED pin as an output:
  pinMode(ledPin, OUTPUT);
  // initialize serial communications:
  Serial.begin(9600);
}

void loop() {
  // read the value of the potentiometer:
  int analogValue = analogRead(analogPin);

  // if the analog value is high enough, turn on the LED:
  if (analogValue > threshold) {
    digitalWrite(ledPin, HIGH);
  } else {
    digitalWrite(ledPin, LOW);
  }

  // print the analog value:
  Serial.println(analogValue);
  delay(1);  // delay in between reads for stability
}
~~~

So i got an error message.

<img src="/images-manual/pixelerror.png" width="1000px">

I dont know what it means so i started looking for diffirent tips on google. So i tried to write a bit of my own and with googles help. i wanted to say if the hour is 14 i will get a message of hello, it's else suck it boys. so i tried this and it works
<img src="/images-manual/if-else.png" width="300px">


Now we can move on to the part where we say when the time is 14:00 the lights go on and the lights will change to a certain color.

# 3. Light on at certain time

so first i tried to look up on google for an Adafruit code to target the LED's. 
I came across File > Examples > Adafruit Neopixel > Simple. i copied this code and put it in my own code.
i defined the pin to D5 and the number of px to 12.

~~~ #include <Adafruit_NeoPixel.h>
#ifdef __AVR__
 #include <avr/power.h> // Required for 16 MHz Adafruit Trinket
#endif

// Which pin on the Arduino is connected to the NeoPixels?
#define PIN        D5 // On Trinket or Gemma, suggest changing this to 1

// How many NeoPixels are attached to the Arduino?
#define NUMPIXELS 12 // Popular NeoPixel ring size

#define DELAYVAL 500 // Time (in milliseconds) to pause between pixels
~~~
I copied this code in de void setup:
~~~
void setup() {
  // These lines are specifically to support the Adafruit Trinket 5V 16 MHz.
  // Any other board, you can remove this part (but no harm leaving it):
#if defined(__AVR_ATtiny85__) && (F_CPU == 16000000)
  clock_prescale_set(clock_div_1);
#endif
  // END of Trinket-specific code.

  pixels.begin(); // INITIALIZE NeoPixel strip object (REQUIRED)
}
~~~

Then i copied this piece of code in my if & else statement:
~~~for(int i=0; i<NUMPIXELS; i++) { // For each pixel...

    // pixels.Color() takes RGB values, from 0,0,0 up to 255,255,255
    // Here we're using a moderately bright green color:
    pixels.setPixelColor(i, pixels.Color(255, 40, 0));

    pixels.show();   // Send the updated pixel colors to the hardware.

    delay(DELAYVAL); // Pause before next pass through loop
  }
  ~~~

- Color in IF statement to ORANGE with RGB
- Color in ELSE statement to PINK with RGB

i uploaded the code and it works.

# 5.light on for multiple hours
the only thing i still have left to do is make sure the lights will stay on for a longer period of time and then switch off in stead of changing in color.

I changed these pieces of code
  ~~~
  ( currentHour == 19, 20, 21, 22, 23, 24)
  ~~~
  
   ~~~
   pixels.setPixelColor(i, pixels.Color(0, 0, 0));
   ~~~
     
Now i uploaded the code but the color is still orange and i wanted it to turn off because it's not 19 hours yes it's 18 hours now. so something is not working.

I tried this code again:
  ~~~
  ( currentHour == 19)
  ~~~
 
 this time it worked. the light turned off. 
 
 so i had to try it in a diffirent way. 
 
 <img src="/images-manual/21error.png" width="1000px">
 
 I tried to make diffirent functions but apparently it doesn't recognize 21 hours. 
 so i removed 21 hours to see if it would work but it doesn't work.
 
 <img src="/images-manual/22error.png" width="1000px">
 
 
 then i tried to copy the whole if statement twice:
   ~~~
   if ( currentHour == 19){
    Serial.println("Hallo het is 17");
    for(int i=0; i<NUMPIXELS; i++) { // For each pixel...

    // pixels.Color() takes RGB values, from 0,0,0 up to 255,255,255
    // Here we're using a moderately bright green color:
    pixels.setPixelColor(i, pixels.Color(255, 40, 0));

    pixels.show();   // Send the updated pixel colors to the hardware.

    delay(DELAYVAL); // Pause before next pass through loop
  }
if ( currentHour == 20){
    Serial.println("Hallo het is 17");
    for(int i=0; i<NUMPIXELS; i++) { // For each pixel...

    // pixels.Color() takes RGB values, from 0,0,0 up to 255,255,255
    // Here we're using a moderately bright green color:
    pixels.setPixelColor(i, pixels.Color(255, 40, 0));

    pixels.show();   // Send the updated pixel colors to the hardware.

    delay(DELAYVAL); // Pause before next pass through loop
  }
 ~~~
    
 But i got another error message. 
 
 <img src="/images-manual/errorhaakje2.png" width="1000px">
     
I added 3 of these brackets "}" at the end

and now it wordss.

I copied the code a couple of times in the if statement and changed the hour by 1+ and now my light works how i want and works.






