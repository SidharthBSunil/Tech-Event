``````


#include <Arduino.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

const char* ssid = "";
const char* password = "";
const char* Gemini_Token = "";
const char* Gemini_Max_Tokens = "100";
String res = "";


void setup() {
  Serial.begin(115200);

  WiFi.mode(WIFI_STA);
  WiFi.disconnect();


  while (!Serial)
    ;


  // wait for WiFi connection
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
}

void loop() 
{
  Serial.println("\nAsk your Question: ");
  while (!Serial.available());

  res = Serial.readStringUntil('\n');
  res.trim();
  res = "\"" + res + "\"";

  Serial.println("Asking Your Question: " + res);

  HTTPClient https;
  if (!https.begin("https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=" + String(Gemini_Token))) {
    Serial.println("[HTTPS] Connection Failed!");
    return;
  }

  https.addHeader("Content-Type", "application/json");
  String payload = "{\"contents\": [{\"parts\":[{\"text\":" + res + "}]}],\"generationConfig\": {\"maxOutputTokens\": " + String(Gemini_Max_Tokens) + "}}";
  int httpCode = https.POST(payload);

  if (httpCode > 0) {
    String response = https.getString();
    DynamicJsonDocument doc(2048);
    
    DeserializationError error = deserializeJson(doc, response);
    if (error) {
      Serial.println("JSON Parsing Failed!");
      return;
    }

    String Answer = doc["candidates"][0]["content"]["parts"][0]["text"];
    Answer.trim();

    Serial.println("\nHere is your Answer:\n" + Answer);
  } else {
    Serial.printf("[HTTPS] Request failed, error: %s\n", https.errorToString(httpCode).c_str());
  }

  https.end();
}
`````
