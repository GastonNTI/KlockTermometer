# KlockTermometer
/*
* Name: clock and temp project
* Author: Gaston Sandegård
* Date: 2024-10-18
* Description: This project uses a ds3231 to measure time and displays the time to an 1306 oled display,
* Further, it measures temprature with a analog temprature module and displays a mapped value to a 9g-servo-motor
*/

// Include Libraries
#include <RTClib.h>         // For DS3231
#include <Wire.h>           // For I2C communication
#include <U8glib.h>         // For OLED display
#include <Servo.h>          // For controlling the servo motor

// Init constants
const int servoPin = 9;     
int redPin= 7;              //Led RGB pins
int greenPin = 6;
int bluePin = 5;

// Init global variables
RTC_DS3231 rtc;
U8GLIB_SSD1306_128X64 u8g(U8G_I2C_OPT_NO_ACK);
Servo myServo;              // Create servo object

// Function declarations
String getTime();
float getTemperature();
void oledWrite(String time, String temperature);
void servoWrite(float value);

void setup() {
    // Init communication
    Serial.begin(9600);
    Wire.begin();

    // Init Hardware
    rtc.begin();
    myServo.attach(servoPin);  // Attach servo to pin 9
    u8g.setFont(u8g_font_unifont); // Set font for OLED
    pinMode(redPin, OUTPUT);
    pinMode(greenPin, OUTPUT);
    pinMode(bluePin, OUTPUT);

    // Optionally adjust the time (if needed)
    // rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
}


void loop() {
    // Get time and temperature
    String timeString = getTime();
    float temperature = getTemperature();  // Get temperature from DS3231

    // Display the time and temperature on OLED but on different lines
    oledWrite(timeString, String(temperature, 1) + " C");

    // Display the temperature in the serial monitor
    Serial.print("Temp: ");
    Serial.print(temperature);
    Serial.println(" °C");

    // Move servo according to temperature
    servoWrite(temperature);


    // Set LED color based on temperature
  if (temperature < 20) {              // Cold
    setColor(0, 0, 255);             // Blue for cold
  } else if (temperature >= 20 && temperature < 25) {  // Moderate
    setColor(0, 255, 0);             // Green for moderate
  } else {                             // Hot
    setColor(255, 0, 0);             // Red for hot
  }


    delay(1000);
}


/*
* This function reads time from the DS3231 module and returns the time as a String.
* Returns: Time in hh:mm:ss format as String
*/

String getTime() {
    DateTime now = rtc.now();  // Get current time

    // Format the time as hh:mm:ss that makes values under 10 write 0 infront
    return String(now.hour() < 10 ? "0" : "") + String(now.hour()) + ":" +
           String(now.minute() < 10 ? "0" : "") + String(now.minute()) + ":" +
           String(now.second() < 10 ? "0" : "") + String(now.second());
}

/*
* This function reads the temperature directly from the DS3231 module.
* Returns temperature in Celsius as float
*/
float getTemperature() {
    return rtc.getTemperature();  // Get temperature from DS3231
}

/*
* This function takes a time and temperature and displays them on the OLED.
* Returns: void
*/

void oledWrite(String time, String temperature) {
    u8g.firstPage();
    do {
        // Draw time on the first line
        u8g.drawStr(0, 20, time.c_str());
        // Draw temperature on the second line
        u8g.drawStr(0, 40, temperature.c_str());
    } while (u8g.nextPage());
}

/*
* This function takes a temperature value and maps it to a corresponding angle
* for a 9g servo motor.
*/
void servoWrite(float temperature) {
    // Define the range for temperature (in Celsius)
    float minTemp = 15;  // Minimum temperature for full range
    float maxTemp = 30;  // Maximum temperature for full range

    // Constrain temperature within the min and max range
    temperature = constrain(temperature, minTemp, maxTemp);

    // Map the temperature range to servo range (0-180 degrees)
    int servoPos = map(temperature * 10, minTemp * 10, maxTemp * 10, 0, 180);

    // Move the servo to the mapped position
    myServo.write(servoPos);

    // Serial Monitor Information
    Serial.print("Servo Position: ");
    Serial.println(servoPos);
}

void setColor(int redValue, int greenValue, int blueValue) { 
  analogWrite(redPin, redValue);
  analogWrite(greenPin, greenValue);
  analogWrite(bluePin, blueValue);
} 
