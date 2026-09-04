<pre>
#include "esp_camera.h"
#include <WiFi.h>
#include <WebServer.h>
#include "FS.h"
#include "SPIFFS.h"

// put wifi stuff here
const char* my_wifi_name = "";
const char* my_wifi_pass = "";

int photonum = 0;
int flashstate = 0;

#define FLASH_LED_PIN 4

#define PWDN_GPIO_NUM     32
#define RESET_GPIO_NUM    -1
#define XCLK_GPIO_NUM      0
#define SIOD_GPIO_NUM     26
#define SIOC_GPIO_NUM     27
#define Y9_GPIO_NUM       35
#define Y8_GPIO_NUM       34
#define Y7_GPIO_NUM       39
#define Y6_GPIO_NUM       36
#define Y5_GPIO_NUM       21
#define Y4_GPIO_NUM       19
#define Y3_GPIO_NUM       18
#define Y2_GPIO_NUM        5
#define VSYNC_GPIO_NUM    25
#define HREF_GPIO_NUM     23
#define PCLK_GPIO_NUM     22

WebServer server(80);

const char INDEX_HTML[] PROGMEM = R"=====(
<!DOCTYPE html>
<html>
<head>
<title>My 3D Printer Monitor Website</title>
<style>
body{
  background-color: #ffeb3b;
  color: #1a1a1a;
  font-family: "Comic Sans MS", sans-serif;
  text-align: center;
}
h1{
  color: #d32f2f;
  font-size: 35px;
}
.box-container{
  background: white;
  border: 5px solid #d32f2f;
  border-radius: 15px;
  padding: 15px;
  max-width: 450px;
  margin: 15px auto;
}
.screen-area{
  background: black;
  border: 3px solid black;
  width: 100%;
}
.screen-area img{
  width: 100%;
}
.cool-button{
  background: #4caf50;
  color: white;
  border: 2px solid black;
  padding: 10px 20px;
  font-size: 16px;
  font-weight: bold;
  border-radius: 8px;
  cursor: pointer;
  margin: 5px;
}
.blue-one{
  background: #2196f3;
}
.orange-one{
  background: #ff9800;
  color: black;
}
.status-bar{
  background: #ff9800;
  padding: 5px;
  border: 1px solid black;
  margin-top: 10px;
}
</style>
</head>
<body>

<h1>🚀 My 3D Printer Cam 🚀</h1>
<p>I made this website to check my printer so it doesnt make plastic spaghetti.</p>

<div class="box-container">
  <h2>Live Video</h2>
  <div class="screen-area">
    <img id="videostream" src="/stream">
  </div>
  <div class="status-bar">Status: <span id="msg">All good!</span></div>
</div>

<div class="box-container">
  <h2>Controls</h2>
  <button class="cool-button orange-one" onclick="clickthelight()">TOGGLE THE FLASH LIGHT!</button>
  <br><br>
  <button class="cool-button" onclick="takesnap()">SNAP A PIC</button>
  <button class="cool-button blue-one" onclick="showsnap()">SHOW PREVIEW</button>
  <br>
  <img id="preview" style="max-width: 150px; margin-top: 10px; display: none; border: 2px solid black;">
</div>

<script>
function clickthelight(){
  fetch('/toggle-light').then(res => res.text()).then(text => {
    document.getElementById('msg').innerText = "Flash is " + text;
  });
}

function takesnap(){
  document.getElementById('msg').innerText = "Snapping image...";
  document.getElementById('videostream').src = "";
  
  fetch('/capture').then(res => res.text()).then(data => {
    document.getElementById('msg').innerText = "Saved picture number " + data;
    setTimeout(function(){ 
      document.getElementById('videostream').src = "/stream?t=" + new Date().getTime(); 
    }, 400);
  });
}

function showsnap(){
  document.getElementById('preview').src = "/get-latest?t=" + new Date().getTime();
  document.getElementById('preview').style.display = "inline-block";
}
</script>

</body>
</html>
)=====";

void handleRoot() {
  server.send(200, "text/html", INDEX_HTML);
}

void handleStream() {
  WiFiClient client = server.client();
  
  client.print("HTTP/1.1 200 OK\r\n");
  client.print("Content-Type: multipart/x-mixed-replace;boundary=xyz_boundary\r\n\r\n");

  while (client.connected()) {
    camera_fb_t * fb = esp_camera_fb_get();
    if (!fb) {
      delay(100);
      continue;
    }
    client.print("\r\n--xyz_boundary\r\n");
    client.printf("Content-Type: image/jpeg\r\nContent-Length: %u\r\n\r\n", fb->len);
    client.write(fb->buf, fb->len);
    esp_camera_fb_return(fb);
    delay(50);
  }
}

void handleCapture() {
  camera_fb_t * fb = esp_camera_fb_get();
  if (!fb) {
    server.send(500, "text/plain", "error");
    return;
  }

  String path = "/pic_" + String(photonum) + ".jpg";
  File f = SPIFFS.open(path, FILE_WRITE);
  
  if (f) {
    f.write(fb->buf, fb->len);
    f.close();
    photonum = photonum + 1;
    server.send(200, "text/plain", String(photonum));
  } else {
    server.send(500, "text/plain", "fail");
  }
  esp_camera_fb_return(fb);
}

void handleGetLatest() {
  if (photonum == 0) {
    server.send(404, "text/plain", "none");
    return;
  }
  int index = photonum - 1;
  String path = "/pic_" + String(index) + ".jpg";
  if (SPIFFS.exists(path)) {
    File f = SPIFFS.open(path, FILE_READ);
    server.streamFile(f, "image/jpeg");
    f.close();
  } else {
    server.send(404, "text/plain", "missing");
  }
}

void handleToggleLight() {
  if (flashstate == 0) {
    flashstate = 1;
    digitalWrite(FLASH_LED_PIN, HIGH);
    server.send(200, "text/plain", "ON");
  } else {
    flashstate = 0;
    digitalWrite(FLASH_LED_PIN, LOW);
    server.send(200, "text/plain", "OFF");
  }
}

void setup() {
  Serial.begin(115200);
  
  pinMode(FLASH_LED_PIN, OUTPUT);
  digitalWrite(FLASH_LED_PIN, LOW);

  SPIFFS.begin(true);

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;
  config.frame_size = FRAMESIZE_VGA;
  config.jpeg_quality = 12;
  config.fb_count = 2;

  esp_camera_init(&config);

  WiFi.begin(my_wifi_name, my_wifi_pass);
  while (WiFi.status() != WL_CONNECTED) { 
    delay(500); 
    Serial.print(".");
  }

  Serial.println("\nconnected!");
  Serial.print("http://");
  Serial.println(WiFi.localIP());

  server.on("/", handleRoot);
  server.on("/stream", handleStream);
  server.on("/capture", handleCapture);
  server.on("/get-latest", handleGetLatest);
  server.on("/toggle-light", handleToggleLight);
  
  server.begin();
}

void loop() {
  server.handleClient();
  delay(2);
}
</pre>
