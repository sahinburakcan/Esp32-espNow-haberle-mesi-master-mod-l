#include <esp_now.h> //esp now kütüphanesi
#include <WiFi.h>   // wifi kütüphanesi
#include <virtuabotixRTC.h>  //rtc(gerçek zamanlı saat)modül kütüphanesi   
#include <LiquidCrystal_I2C.h> //lcd kütüphanesi

int CLK_PIN = 16;      //rtc için kontrol pinleri tanımı                                 
int DAT_PIN = 17;                                        
int RST_PIN = 26; 

virtuabotixRTC myRTC(CLK_PIN, DAT_PIN, RST_PIN); //rtc kontrol pinleri

int lcdColumns = 16; //lcd ekran boyut tanımlaması
int lcdRows = 2;//lcd ekran boyut tanımlaması

LiquidCrystal_I2C lcd(0x27, lcdColumns, lcdRows);//lcd ekran boyut tanımlaması

// mac adreslerinin tanımlanması
uint8_t broadcastAddress1[] = {0x9C, 0x9C, 0x1F, 0x47, 0xC9, 0x32};
uint8_t broadcastAddress2[] = {0x9C, 0x9C, 0x1F, 0x47, 0xB6, 0x6D};

typedef struct test_struct { //struct yapıda int değişken tanımlamsı
  int x;
  int y;
} test_struct;

test_struct test;

// gönderim yapıldığında geri bildirim yapacak komutlar
void OnDataSent(const uint8_t *mac_addr, esp_now_send_status_t status) {
  char macStr[18];
  Serial.print("Packet to: ");
  // Copies the sender mac address to a string
  snprintf(macStr, sizeof(macStr), "%02x:%02x:%02x:%02x:%02x:%02x",
           mac_addr[0], mac_addr[1], mac_addr[2], mac_addr[3], mac_addr[4], mac_addr[5]);
  Serial.print(macStr);
  Serial.print(" send status:\t");
  Serial.println(status == ESP_NOW_SEND_SUCCESS ? "Delivery Success" : "Delivery Fail");
}
 
void setup() {
  Serial.begin(115200);// seri haberleşme başlangıcı

   lcd.init();// lcd başlangıcı
   lcd.backlight();// lcd başlangıcı
 
  WiFi.mode(WIFI_STA);//wifi istasyonu olarak belirleme
 
  if (esp_now_init() != ESP_OK) {//esp now başlangıcı
    Serial.println("Error initializing ESP-NOW");
    return;
  }
  
  esp_now_register_send_cb(OnDataSent);//OnDataSent tanımlaması
   
  // cihaz eşleştirme için ayarlar
  esp_now_peer_info_t peerInfo;
  peerInfo.channel = 0;  
  peerInfo.encrypt = false;
  
  //1. chaz eşleştirmesi 
  memcpy(peerInfo.peer_addr, broadcastAddress1, 6);
  if (esp_now_add_peer(&peerInfo) != ESP_OK){
    Serial.println("Failed to add peer");
    return;
  }
  //2. chaz eşleştirmesi  
  memcpy(peerInfo.peer_addr, broadcastAddress2, 6);
  if (esp_now_add_peer(&peerInfo) != ESP_OK){
    Serial.println("Failed to add peer");
    return;
  }
}
 
void loop() { 
   myRTC.updateTime(); //rtc başlangıcı
int a, b;
 Serial.println(test.x);
 if(myRTC.hours>=16 && myRTC.hours<=6){ //oto mod için sorgu 
 test.x=1;//oto mod değişken ayarlaması
 }
 else{
 test.x=0;//oto mod değişken ayarlaması
 } 
 lcd.setCursor(0,0);//lcd ekran ayarlaması
lcd.print("deger ayarlayiniz");
b=analogRead(35);// pot değerinin okunması
      Serial.println(b);
  lcd.setCursor(0,1);
  lcd.print(b);
  if(b<=300){//manuel mod için sorgu
  test.y=0;//manuel mod değişken ayarlaması
  
}else{
  test.y=b;//manuel mod değişken ayarlaması
}
Serial.print("y=");
Serial.println(test.y);

//ayıtlı mac adreslerine veri gönderimi
  esp_err_t result = esp_now_send(0, (uint8_t *) &test, sizeof(test_struct));
   //veri gönderim kontrolü
  if (result == ESP_OK) {
    Serial.println("Sent with success");
  }
  else {
    Serial.println("Error sending the data");
  }
  delay(2000);
}
 
