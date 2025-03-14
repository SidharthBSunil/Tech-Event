```
#include <Arduino.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include "Audio.h" // Added for audio functionality

#define I2S_DOUT      25
#define I2S_BCLK      27
#define I2S_LRC       26

const char* ssid = "SSID";
const char* password = "PASS";
const char* Gemini_Token = "GEMINI_API_KEY";
const char* Gemini_Max_Tokens = "100";
String res = "";

Audio audio; // Audio object

void setup() {
  Serial.begin(115200);

  WiFi.mode(WIFI_STA);
  WiFi.disconnect();

  while (!Serial)
    ;

  WiFi.begin(ssid, password);
  Serial.print("Connecting to ");
  Serial.println(ssid);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT);
  audio.setVolume(100);
}

void loop() {
  audio.loop(); // Keep audio running

  Serial.println("");
  Serial.println("Ask your Question : ");
  while (Serial.available() == 0) {
    delay(10);
  }
  res = "";
  while (Serial.available() > 0) {
    char add = Serial.read();
    if (add == '\n' || add == '\r') {
      break;
    }
    res += add;
  }

  res = "\"" + res + "\"";
  Serial.println("");
  Serial.print("Asking Your Question : ");
  Serial.println(res);

  HTTPClient https;

  if (https.begin("https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=" + (String)Gemini_Token)) {
    https.addHeader("Content-Type", "application/json");
    String payload = String("{\"contents\": [{\"parts\":[{\"text\":" + res + "}]}],\"generationConfig\": {\"maxOutputTokens\": " + (String)Gemini_Max_Tokens + "}}");

    int httpCode = https.POST(payload);

    if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_MOVED_PERMANENTLY) {
      String payload = https.getString();

      DynamicJsonDocument doc(1024);
      deserializeJson(doc, payload);
      String Answer = doc["candidates"][0]["content"]["parts"][0]["text"];

      Answer.trim();
      String filteredAnswer = "";
      for (size_t i = 0; i < Answer.length(); i++) {
        char c = Answer[i];
        if (isalnum(c) || isspace(c)) {
          filteredAnswer += c;
        } else {
          filteredAnswer += ' ';
        }
      }
      Answer = filteredAnswer;

      Serial.println("");
      Serial.println("Here is your Answer: ");
      Serial.println("");
      Serial.println(Answer);

      // Convert and play the answer using audio library
      audio.connecttospeech(Answer.c_str(), "en"); // Play the answer
    } else {
      Serial.printf("[HTTPS] GET... failed, error: %s\n", https.errorToString(httpCode).c_str());
    }
    https.end();
  } else {
    Serial.printf("[HTTPS] Unable to connect\n");
  }
  res = "";
}

void audio_info(const char *info) {
  Serial.print("audio_info: ");
  Serial.println(info);
}
```
