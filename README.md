# Test2
hello world test
#include <ESP8266WiFi.h>

// تنظیمات شبکه Wi-Fi
const char* ssid = "YOUR_SSID"; // نام شبکه Wi-Fi خود را وارد کنید
const char* password = "YOUR_PASSWORD"; // رمز عبور شبکه Wi-Fi خود را وارد کنید

// پین‌های LED RGB
int redPin = 9;
int greenPin = 10;
int bluePin = 11;

// ایجاد شیء سرور
WiFiServer server(80);

void setup() {
  // راه‌اندازی پین‌ها
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);

  // راه‌اندازی Wi-Fi
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  
  Serial.println("Connected to WiFi");
  server.begin();
}

void loop() {
  WiFiClient client = server.available(); // بررسی اتصال کلاینت

  if (client) {
    String currentLine = ""; // خط جاری
    while (client.connected()) {
      if (client.available()) {
        char c = client.read(); // خواندن داده‌ها
        Serial.write(c); // نمایش در سریال مانیتور

        // اگر خط جدیدی دریافت شود
        if (c == '\n') {
          // بررسی درخواست GET
          if (currentLine.length() == 0) {
            // ارسال پاسخ HTTP
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println();

            // HTML برای کنترل رنگ LED
            client.println("<html><body>");
            client.println("<h1>Control RGB LED</h1>");
            client.println("<form action="/setColor" method="GET">");
            client.println("Red: <input type="range" name="r" min="0" max="255"><br>");
            client.println("Green: <input type="range" name="g" min="0" max="255"><br>");
            client.println("Blue: <input type="range" name="b" min="0" max="255"><br>");
            client.println("<input type="submit" value="Set Color">");
            client.println("</form>");
            client.println("</body></html>");
            break;
          } else {
            // پردازش درخواست GET برای تنظیم رنگ
            if (currentLine.startsWith("GET /setColor")) {
              int r = getValue(currentLine, "r");
              int g = getValue(currentLine, "g");
              int b = getValue(currentLine, "b");

              // تنظیم رنگ LED
              analogWrite(redPin, r);
              analogWrite(greenPin, g);
              analogWrite(bluePin, b);
            }
          }
          currentLine = ""; // خط جاری را خالی کن
        } else {
          currentLine += c; // اضافه کردن کاراکتر به خط جاری
        }
      }
    }
    client.stop(); // قطع اتصال
  }
}

int getValue(String line, String name) {
  int startIndex = line.indexOf(name + "=");
  if (startIndex == -1) return 0;
  
  startIndex += name.length() + 1;
  int endIndex = line.indexOf("&", startIndex);
  
  if (endIndex == -1) endIndex = line.length();
  
  return line.substring(startIndex, endIndex).toInt();
}
