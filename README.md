# AI_SMART_HOME
🧠 SYSTEM YAWE IZAKORA ITE? 📡 ESP32  Izakora:  WiFi Access Point Smart home control LED / Servo / Sensors Web server 📱 Phone Browser  Ni yo izakora:  🎤 microphone 🔊 voice output 🧠 AI processing speec
/*
========================================================
        AI SMART HOME - INTERNET VERSION
========================================================

FEATURES:
✅ ESP32 connects to your hotspot
✅ AI Voice Commands
✅ Smart Home Dashboard
✅ DHT11 Sensor
✅ Servo Door
✅ 2 LEDs
✅ Female Voice Responses
✅ Chrome Browser AI
✅ Internet Ready

========================================================
*/

#include <WiFi.h>
#include <WebServer.h>
#include <DHT.h>
#include <ESP32Servo.h>

// =====================================================
// YOUR PHONE HOTSPOT
// =====================================================
const char* wifiName = "AI_SMART_HOME";
const char* wifiPass = "12345678";

// =====================================================
// STATIC IP
// =====================================================
IPAddress local_IP(192,168,43,200);

IPAddress gateway(192,168,43,1);

IPAddress subnet(255,255,255,0);

// =====================================================
// DHT11
// =====================================================
#define DHTPIN 4
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

// =====================================================
// LEDs
// =====================================================
#define LIGHT1 26
#define LIGHT2 27

// =====================================================
// SERVO
// =====================================================
#define SERVO_PIN 13

Servo doorServo;

// =====================================================
// SERVER
// =====================================================
WebServer server(80);

// =====================================================
// HTML PAGE
// =====================================================
String webpage = R"rawliteral(

<!DOCTYPE html>
<html>
<head>

<meta name="viewport"
content="width=device-width, initial-scale=1">

<title>AI SMART HOME</title>

<link rel="stylesheet"
href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">

<style>

*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Arial;
}

body{
background:linear-gradient(135deg,#020617,#0f172a);
color:white;
padding:20px;
}

.title{
text-align:center;
font-size:40px;
font-weight:bold;
margin-bottom:20px;
}

.grid{
display:grid;
grid-template-columns:repeat(auto-fit,minmax(250px,1fr));
gap:20px;
}

.card{
background:rgba(255,255,255,0.08);
padding:25px;
border-radius:25px;
backdrop-filter:blur(10px);
box-shadow:0 8px 25px rgba(0,0,0,0.3);
}

.icon{
font-size:50px;
margin-bottom:15px;
}

.tempIcon{color:#ff6b6b;}
.lightIcon{color:#ffd43b;}
.doorIcon{color:#63e6be;}

.value{
font-size:40px;
font-weight:bold;
}

button{
width:100%;
padding:15px;
margin-top:15px;
border:none;
border-radius:15px;
font-size:18px;
font-weight:bold;
cursor:pointer;
background:#22c55e;
color:white;
}

button:hover{
transform:scale(1.03);
}

#aiResponse{
margin-top:20px;
font-size:20px;
color:#4ade80;
text-align:center;
}

.spin{
animation:spin 1s linear infinite;
}

@keyframes spin{
from{transform:rotate(0deg);}
to{transform:rotate(360deg);}
}

</style>

</head>

<body>

<div class="title">
🤖 AI SMART HOME
</div>

<div class="grid">

<!-- TEMP -->
<div class="card">

<div class="icon tempIcon">
<i class="fa-solid fa-temperature-high"></i>
</div>

Temperature

<div class="value" id="temp">
-- °C
</div>

</div>

<!-- LIGHT 1 -->
<div class="card">

<div class="icon lightIcon">
<i id="light1Icon"
class="fa-solid fa-lightbulb"></i>
</div>

Living Room Light

<button onclick="light1on()">
TURN ON
</button>

<button onclick="light1off()">
TURN OFF
</button>

</div>

<!-- LIGHT 2 -->
<div class="card">

<div class="icon lightIcon">
<i id="light2Icon"
class="fa-solid fa-lightbulb"></i>
</div>

Bedroom Light

<button onclick="light2on()">
TURN ON
</button>

<button onclick="light2off()">
TURN OFF
</button>

</div>

<!-- DOOR -->
<div class="card">

<div class="icon doorIcon">
<i class="fa-solid fa-door-open"></i>
</div>

Smart Door

<button onclick="openDoor()">
OPEN
</button>

<button onclick="closeDoor()">
CLOSE
</button>

</div>

<!-- AI -->
<div class="card">

<div class="icon">
🎤
</div>

AI Voice Assistant

<button onclick="startListening()">
START TALKING
</button>

</div>

</div>

<div id="aiResponse">
Waiting for voice command...
</div>

<script>

// =====================================
// FEMALE VOICE
// =====================================
function speak(text){

let speech =
new SpeechSynthesisUtterance(text);

speech.rate = 1;
speech.pitch = 1.1;

let voices =
speechSynthesis.getVoices();

for(let voice of voices){

if(voice.name.includes("Female") ||
voice.name.includes("Google UK English Female")){

speech.voice = voice;

}

}

speechSynthesis.speak(speech);

}

// =====================================
// SPEECH RECOGNITION
// =====================================
const recognition =
new(window.SpeechRecognition ||
window.webkitSpeechRecognition)();

recognition.lang = "en-US";

// =====================================
// START LISTENING
// =====================================
function startListening(){

recognition.start();

document.getElementById("aiResponse")
.innerHTML = "Listening...";

}

// =====================================
// COMMANDS
// =====================================
recognition.onresult = function(event){

let command =
event.results[0][0].transcript.toLowerCase();

document.getElementById("aiResponse")
.innerHTML = command;

// LIGHT 1
if(command.includes("turn on light one")){

light1on();

speak("Light one turned on");

}

else if(command.includes("turn off light one")){

light1off();

speak("Light one turned off");

}

// LIGHT 2
else if(command.includes("turn on light two")){

light2on();

speak("Light two turned on");

}

else if(command.includes("turn off light two")){

light2off();

speak("Light two turned off");

}

// DOOR
else if(command.includes("open the door")){

openDoor();

speak("Door opened");

}

else if(command.includes("close the door")){

closeDoor();

speak("Door closed");

}

// TEMPERATURE
else if(command.includes("temperature")){

fetch("/temp")
.then(response => response.text())
.then(data => {

speak("The temperature is " +
data + " degrees Celsius");

});

}

else{

speak("Sorry, I did not understand");

}

};

// =====================================
// LIGHT FUNCTIONS
// =====================================
function light1on(){

fetch("/light1on");

document.getElementById("light1Icon")
.style.color="#ffd43b";

}

function light1off(){

fetch("/light1off");

document.getElementById("light1Icon")
.style.color="white";

}

function light2on(){

fetch("/light2on");

document.getElementById("light2Icon")
.style.color="#ffd43b";

}

function light2off(){

fetch("/light2off");

document.getElementById("light2Icon")
.style.color="white";

}

// =====================================
// DOOR
// =====================================
function openDoor(){

fetch("/openDoor");

speak("Door opened");

}

function closeDoor(){

fetch("/closeDoor");

speak("Door closed");

}

// =====================================
// LIVE TEMP
// =====================================
setInterval(() => {

fetch("/temp")
.then(response => response.text())
.then(data => {

document.getElementById("temp")
.innerHTML = data + " °C";

});

},2000);

</script>

</body>
</html>

)rawliteral";

// =====================================================
// ROOT
// =====================================================
void handleRoot(){

server.send(200,"text/html",webpage);

}

// =====================================================
// TEMP
// =====================================================
void handleTemp(){

float t = dht.readTemperature();

server.send(200,"text/plain",String(t));

}

// =====================================================
// LIGHTS
// =====================================================
void light1on(){

digitalWrite(LIGHT1,HIGH);

server.send(200,"text/plain","OK");

}

void light1off(){

digitalWrite(LIGHT1,LOW);

server.send(200,"text/plain","OK");

}

void light2on(){

digitalWrite(LIGHT2,HIGH);

server.send(200,"text/plain","OK");

}

void light2off(){

digitalWrite(LIGHT2,LOW);

server.send(200,"text/plain","OK");

}

// =====================================================
// DOOR
// =====================================================
void openDoor(){

doorServo.write(90);

server.send(200,"text/plain","OPEN");

}

void closeDoor(){

doorServo.write(0);

server.send(200,"text/plain","CLOSE");

}

// =====================================================
// SETUP
// =====================================================
void setup(){

Serial.begin(115200);

// OUTPUTS
pinMode(LIGHT1,OUTPUT);
pinMode(LIGHT2,OUTPUT);

digitalWrite(LIGHT1,LOW);
digitalWrite(LIGHT2,LOW);

// DHT
dht.begin();

// SERVO
doorServo.attach(SERVO_PIN);
doorServo.write(0);

// STATIC IP
WiFi.config(local_IP, gateway, subnet);

// CONNECT WIFI
WiFi.begin(wifiName,wifiPass);

Serial.print("Connecting");

while(WiFi.status() != WL_CONNECTED){

delay(500);
Serial.print(".");

}

Serial.println("");
Serial.println("CONNECTED!");

Serial.print("ESP32 IP: ");
Serial.println(WiFi.localIP());

// ROUTES
server.on("/",handleRoot);

server.on("/temp",handleTemp);

server.on("/light1on",light1on);
server.on("/light1off",light1off);

server.on("/light2on",light2on);
server.on("/light2off",light2off);

server.on("/openDoor",openDoor);
server.on("/closeDoor",closeDoor);

// START SERVER
server.begin();

Serial.println("SERVER STARTED");

}

// =====================================================
// LOOP
// =====================================================
void loop(){

server.handleClient();

}
