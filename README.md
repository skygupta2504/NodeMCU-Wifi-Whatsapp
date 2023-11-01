# NodeMCU-Wifi-Whatsapp
//Arduino IDE code for NodeMCU to get the data from the Arduino UNO &amp; to connect to the user's WIFI and send messages to the user's registered Whatsapp number using the Whatsapp chatbot. Also sending the data to Thingspeak channel.
//NodeMCU code
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include<ESP8266WebServer.h>
#include <WiFiClient.h>
#include <UrlEncode.h>
#include <SoftwareSerial.h>
#include <ArduinoJson.h>
int val=0;
double field1 =0.0;
const char* ssid = "Wifi_name";
const char* password = "Wifi_password";
const char * host ="api.thingspeak.com";
// +international_country_code + phone number
// Portugal +351, example: +351912345678
String phoneNumber = "phone_no.";
String apiKey = "sent_from_whatsapp_chatbot";
String apikey1="Thingspeak_channel_apikey";
void sendMessage(String message){


  String url = "http://api.callmebot.com/whatsapp.php?phone=" + phoneNumber + "&apikey=" + apiKey + "&text=" + urlEncode(message);
  WiFiClient client; 
  WiFiClient client2; 
  WiFiClient client3; 
  HTTPClient http;
  http.begin(client, url);

  // Specify content-type header
  http.addHeader("Content-Type", "application/x-www-form-urlencoded");
  
  // Send HTTP POST request
  int httpResponseCode = http.POST(url);
  if (httpResponseCode == 200){
    Serial.print("Message sent successfully");
  }
  else{
    Serial.println("Error sending the message");
    Serial.print("HTTP response code: ");
    Serial.println(httpResponseCode);
  }

  // Free resources
  http.end();
}
//Include Lib for Arduino to Nodemcu


//D6 = Rx & D5 = Tx
SoftwareSerial nodemcu(D6, D5);


void setup() {
  
  Serial.begin(9600);
  nodemcu.begin(9600);
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  Serial.println("Connecting");
  //Wait for connection
  while(WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("conected to ");
  Serial.println(ssid);
  Serial.print("IP address");
  Serial.println(WiFi.localIP());


    
}

void loop() {
  
  StaticJsonBuffer<1000> jsonBuffer;
      JsonObject& data = jsonBuffer.parseObject(nodemcu);

  if (data == JsonObject::invalid()) {
        //Serial.println("Invalid Json Object");
        jsonBuffer.clear();
        return;
      }

   Serial.println("JSON Object Recieved");
      Serial.print("Recieved FlowDiff:  ");
      double diff = data["flowDifference"];

   double l_minute2=data["flowrate1"];
      double l_minute3=data["flowrate2"];
      Serial.println(abs(diff));
      Serial.println(l_minute2);
       Serial.println(l_minute3);
      if(abs(diff)>=0.04){
        val++;
        if(val>3){
          sendMessage("Water leakage detected");
          val=0;
          }

  }
      else val=0;
    delay(500);
      Serial.println("-----------------------------------------");
      
   WiFiClient client;
    WiFiClient client1;
    WiFiClient client2;
    const int httpPort = 80;

    //Connect to host, host (web site) is define at top
   if(!client.connect (host, httpPort)) {
    Serial.println("Connection Failed");
    delay(300);
    return; //Keep retrying until we get connected
    }

      //Field1
  String ADCData;
    double adcvalue=diff;
    ADCData = String (adcvalue); //String to interger conversion
 
    
    //Reading the value of the analog POT Sensor /String to interger conversion
   String Link1="GET /update?api_key="+apikey1+"&field1="; 
    Link1 = Link1 + ADCData;
    Link1 = Link1 + " HTTP/1.1\r\n" + "Host: " + host + "\r\n" + "Connection: close\r\n\r\n";
    client.print(Link1);
    delay (100);

//Field 2
    String ADCData2;
    double adcvalue2=l_minute3;
    ADCData2 = String (adcvalue2); //String to interger conversion
    // string Link="GET /update?api_key="+apiKey+" & fieldl="; 
    
    //Reading the value of the analog POT Sensor /String to interger conversion
   String Link2="GET /update?api_key="+apikey1+"&field2="; 
    Link2 = Link2 + ADCData2;
    Link2 = Link2 + " HTTP/1.2\r\n" + "Host: " + host + "\r\n" + "Connection: close\r\n\r\n";
    client1.print(Link2);
    delay (100);
//Field 3
    String ADCData3;
    double adcvalue3=diff;
    ADCData3 = String (adcvalue3); //String to interger conversion
    // string Link="GET /update?api_key="+apiKey+" & fieldl="; 
    
    //Reading the value of the analog POT Sensor /String to interger conversion
  String Link="GET /update?api_key="+apikey1+"&field3="; 
    Link = Link + ADCData3;
    Link = Link + " HTTP/1.1\r\n" + "Host: " + host + "\r\n" + "Connection: close\r\n\r\n";
    client2.print(Link);
    delay (100);


    // Wait for server to respond with vineout of 5 Seconds
  int timeout=0;
    while((!client.available()) && (timeout < 1000)){
    delay (10);
    timeout = timeout+1;
    }
    if(timeout<500){
      while(client.available()){
        Serial.println(client.readString());
      }
    }
    else{
      Serial.println("Request Timeout");
    }
    delay(500);
}
