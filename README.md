# wokwiesp32servo_code
#include <WiFi.h>
#include <FirebaseESP32.h>
#include <DHT.h>
#include <ESP32Servo.h>

#define DHTPIN 2
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);
int led1 = 5;
String zero = "0", one = "1";
String Led_Stat;
String Servo_Stat;
const int servoPin = 4;
Servo servo;

// Mendeklarasikan Koneksi Wifi dan firebase

#define FIREBASE_HOST "https://led-controll-97565-default-rtdb.firebaseio.com/" // Isi Dengan Firebase Host Anda

#define FIREBASE_AUTH"UEOkEWebXsVDJXYXuKlx4Pd2JCkDhNoZa099kGxB" // Isi Dengan Firebase Host Anda
#define WIFI_SSID "Wokwi-GUEST" // Ganti dengan ssid wifi jika tidak ingin menggunakan wokwi
#define WIFI_PASSWORD "" // Ganti dengan password wifi jika tidak ingin menggunakan wokwi

// Mendeklarasikan objek data dari FirebaseESP8266
FirebaseData firebaseData;

void setup() {

  Serial.begin(115200);

  pinMode(led1, OUTPUT);

  dht.begin();

  // Koneksi ke Wifi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("connecting");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }
  Serial.println();
  Serial.print("Connected with IP: ");

  Serial.println(WiFi.localIP());
  Serial.println();

  Firebase.begin(FIREBASE_HOST, FIREBASE_AUTH);
  Firebase.setString(firebaseData, "/data_sensor/status_led", zero);
  Firebase.setString(firebaseData, "/data_sensor/status_servo", zero);
}

int pos = 0;

void loop() {

  // Sensor membaca suhu dan kelembaban
  float t = dht.readTemperature();
  float h = dht.readHumidity();
  float n = 0;

  // Memeriksa apakah sensor berhasil mambaca suhu dankelembaban
  if (isnan(t) || isnan(h)) {
    Serial.println("Gagal membaca sensor");
    return;
  }

  servo.attach(servoPin, 500, 2400);

  // Menampilkan suhu dan kelembaban pada serial monitor
  Serial.print("Suhu: ");
  Serial.print(t);
  Serial.println(" *C");
  Serial.print("Kelembaban: ");
  Serial.print(h);
  Serial.println(" %");
  Serial.println();

  // Mengirim status suhu dan kelembaban ke firebase
  if (Firebase.setFloat(firebaseData, "/data_sensor/suhu", t)) {
    Serial.println("Suhu terkirim");
  } else {
    Serial.println("Suhu tidak terkirim");
    Serial.println("Karena: " + firebaseData.errorReason());
  }


  if (Firebase.setFloat(firebaseData, "/data_sensor/kelembaban", h)) {
    Serial.println("Kelembaban terkirim");
    Serial.println();
  } else {
    Serial.println("Kelembaban tidak terkirim");
    Serial.println("Karena: " + firebaseData.errorReason());
  }

  // Mengontrol led
  Firebase.getString(firebaseData, "/data_sensor/status_led");
  Led_Stat = firebaseData.stringData(); Serial.print('\n');
  if (Led_Stat == one) {
    digitalWrite(led1, HIGH);
  } else if (Led_Stat == zero) {
    digitalWrite(led1, LOW);
  } else {
    Serial.println("Command Error!");
  }

  // Mengontrol Servo
  Firebase.getString(firebaseData, "/data_sensor/status_servo");
  Servo_Stat = firebaseData.stringData();
  Serial.print('\n');
  if (Servo_Stat == one) {
    for (pos = 0; pos <= 180; pos += 1) {
      servo.write(pos);
      delay(10);
    }
    for (pos = 180; pos >= 0; pos -= 1) {
      servo.write(pos);
      delay(10);
    }
    delay(500);
    Firebase.setFloat(firebaseData, "/data_sensor/status_servo", n);
  } else if (Servo_Stat == zero) {
    digitalWrite(led1, LOW);
  } else {
    Serial.println("Command Error!");
  }
}
