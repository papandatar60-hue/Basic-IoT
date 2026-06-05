#define BLYNK_TEMPLATE_ID "ISI_TEMPLATE_ID"
#define BLYNK_TEMPLATE_NAME "ISI_TEMPLATE_NAME"
#define BLYNK_AUTH_TOKEN "ISI_AUTH_TOKEN"

#define BLYNK_PRINT Serial

#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DHT.h>

// WiFi
char ssid[] = "NAMA_WIFI";
char pass[] = "PASSWORD_WIFI";

// Pin sensor dan aktuator
#define MQ2_PIN A0
#define DHT_PIN D7
#define BUZZER_PIN D5
#define LED_PIN D1

#define DHTTYPE DHT22
DHT dht(DHT_PIN, DHTTYPE);

BlynkTimer timer;

// ================= THRESHOLD =================
int gasThreshold = 1000;          // Batas gas MQ2
float suhuThreshold = 32.0;       // Batas suhu maksimum
float kelembapanMin = 20.0;       // Batas kelembapan minimum
float kelembapanMax = 80.0;       // Batas kelembapan maksimum

// Tone buzzer
int buzzerTone = 1000;

// ================= VIRTUAL PIN BLYNK =================
// V0 = Suhu
// V1 = Kelembapan
// V2 = Nilai MQ2
// V3 = Status Gas
// V4 = Status Suhu
// V5 = Status Kelembapan

void buzzerBahaya() {
  tone(BUZZER_PIN, buzzerTone);
}

void buzzerMati() {
  noTone(BUZZER_PIN);
  digitalWrite(BUZZER_PIN, LOW);
}

void bacaSensor() {
  int gasValue = analogRead(MQ2_PIN);

  float suhu = dht.readTemperature();
  float kelembapan = dht.readHumidity();

  if (isnan(suhu) || isnan(kelembapan)) {
    Serial.println("Gagal membaca sensor DHT22!");

    Blynk.virtualWrite(V4, "Sensor Suhu Error");
    Blynk.virtualWrite(V5, "Sensor Kelembapan Error");

    return;
  }

  Serial.println("===== DATA SENSOR =====");

  Serial.print("Gas MQ2: ");
  Serial.println(gasValue);

  Serial.print("Suhu: ");
  Serial.print(suhu);
  Serial.println(" °C");

  Serial.print("Kelembapan: ");
  Serial.print(kelembapan);
  Serial.println(" %");

  // Kirim nilai sensor ke Blynk
  Blynk.virtualWrite(V0, suhu);
  Blynk.virtualWrite(V1, kelembapan);
  Blynk.virtualWrite(V2, gasValue);

  // Cek kondisi threshold
  bool gasBahaya = gasValue > gasThreshold;
  bool suhuBahaya = suhu > suhuThreshold;
  bool kelembapanBahaya = kelembapan < kelembapanMin || kelembapan > kelembapanMax;

  // ================= STATUS GAS KE V3 =================
  if (gasBahaya) {
    Blynk.virtualWrite(V3, "Bahaya! Gas/Asap Terdeteksi");
    Serial.println("Status Gas: Bahaya");
  } else {
    Blynk.virtualWrite(V3, "Aman");
    Serial.println("Status Gas: Aman");
  }

  // ================= STATUS SUHU KE V4 =================
  if (suhuBahaya) {
    Blynk.virtualWrite(V4, "Suhu Tinggi");
    Serial.println("Status Suhu: Suhu Tinggi");
  } else {
    Blynk.virtualWrite(V4, "Suhu Normal");
    Serial.println("Status Suhu: Suhu Normal");
  }

  // ================= STATUS KELEMBAPAN KE V5 =================
  if (kelembapanBahaya) {
    Blynk.virtualWrite(V5, "Kelembapan Tidak Normal");
    Serial.println("Status Kelembapan: Tidak Normal");
  } else {
    Blynk.virtualWrite(V5, "Kelembapan Normal");
    Serial.println("Status Kelembapan: Normal");
  }

  // ================= OUTPUT GAS KE BUZZER =================
  if (gasBahaya) {
    buzzerBahaya();
    Serial.println("Output Gas: Buzzer Menyala");
  } else {
    buzzerMati();
    Serial.println("Output Gas: Buzzer Mati");
  }

  // ================= OUTPUT SUHU & KELEMBAPAN KE LED =================
  if (suhuBahaya || kelembapanBahaya) {
    digitalWrite(LED_PIN, HIGH);
    Serial.println("Output Suhu/Kelembapan: LED Menyala");
  } else {
    digitalWrite(LED_PIN, LOW);
    Serial.println("Output Suhu/Kelembapan: LED Mati");
  }

  Serial.println();
}

void setup() {
  Serial.begin(115200);

  pinMode(MQ2_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);

  digitalWrite(LED_PIN, LOW);
  buzzerMati();

  dht.begin();

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  // Membaca sensor setiap 2 detik
  timer.setInterval(2000L, bacaSensor);
}

void loop() {
  Blynk.run();
  timer.run();
}
