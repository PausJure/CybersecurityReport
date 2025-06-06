# Developing an ESP32 EvilTwin attack tool using ChatGPT

## Preface
![Photo of Mountain](images/mountain.jpg)

With the rise of LLMs many things changed. One of those things is coding; users that previously didn't know how to code can now use natural language to prompt LLMs to generate relatively complex code. This code more often than not doesn't comply with industry standards in terms of quality/security but it is much better than what someone with basic knowledge in coding could do by himself. (Not to mention the time saved)

This Phenomenon is called **"Vibe coding"**. 

This brings us to the goal of this report: A student with general, basic knowledge in coding will bild an attack tool and (hopefully) demonstrate how chaining the use of **"exploiting" LLMs for malicious purposes** and the **"vulnerabilities" present in people**, could allow adversaries access to public or industry infrastructure with potentialy catastrophic results. 


## Introduction
The idea is to leverage the use of ChatGPT and build a generic EvilTwin attack tool. To do so we will first focus on building it for a specific purpose, in this case attacking UniTS. We willl then proceede with generalizing the code and instructions to craeate a general tool for ESP32 devices. 

We will exploit the fact that users often get random disconnections from eduroam and try to connect manually to eduroam. 
This is most noticable in areas near the diningh hall as eduroam doesn't fully cover it, or during class where professors under the pressure of time try to connect to the eduroam wifi unsuccesfully, only to give up and choose a wired connection and getting prompted to login while simultaneously blaming the university or eduroam itself for the unneccessary authentication checkup. 

It is this last experience where this idea came from. 
We will create a tool that spowns an evil AP and saves the imput from the captive portal seprately. From personal experience, no one, with the exception of experienced individals would notice it was an attacker as they would most certainly blame the university or eduroam itself for the fact that the login didn't work (if they even notice it didn't).


## DETAILED PLAN

The idea is to create a generic tool for performing evil twin attacks using the ESP32 microcontroller. To do so we will firs "exploit" a LLM (in this case ChatGPT) to get a working code that performs an evil twin attack on the University of Trieste that we will later generalize and build the necessary documentation for using the tool.

To program the ESP32 microcontroller we will use the Arduino IDE (2.3.6) as it is an easy and effective way to program the ESP32. We will also need to use the Legacy IDE (Arduino IDE 1.8.19) as we will need a tool called SPIFFS which isn't compatible with the latest versions of arduino IDE. We could use just the legycy i+IDE but the new one is faster and more powerful thus we will mostly use that one to compile and push our code to the ESP32.

As we will be "vibe coding" we need to have a clear idea of what we want to do while also taking into consideration what can be done. This is because we dont want to let the LLM lead the programming flow and thus deviating and messing up the code more than it should (hallucination effect). It works best when we have a clear goal. As stated before we also need to take into consideration what can be done: the ESP32 is a cheap microcontroller and it cannot do complex tasks. 

What we cannot do :  
+ Host large websites
+ Perform deauthentication using management frames, since eduroam seems to  use WPA3 enterprise or WPA2 enterprise (for older devices) so they probably use PMF (protected management frames) as they are mandatory in WPA3 enterprise while in WPA2 they can be toggled on or off but we will assume they are on so the ESP32 has no chance in forging those management frames.

Taking this into consideration, we will ask chatgpt to create lightweight copies of the login html pages used by the university APs. And we won't try to use deauthentication* but we will instead leverage the random disconnections and limited coverage of the eduroam network on the campus.


* see STEP 8


Short summary of the process: 

+ create a code that will make an AP with ssid "eduroam" on the ESP32 device
+ make the code prompt the users to "login to use this network" by a push notification
+ redirect to a login webpage very simmilar to the Units Eventi Wifi login page
+ save the login data onto a separate webpage hosted on the esp32 for easy access
+ give a "connection successful" notice to the victim
+ generalize the code and build the documentation to create a usable tool


  
