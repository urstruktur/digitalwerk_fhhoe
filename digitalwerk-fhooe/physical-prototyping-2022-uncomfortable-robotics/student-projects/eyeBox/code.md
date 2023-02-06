---
title: Code
layout: default
parent: eyeBox
nav_order: 1
---

# Code description
**ESP32 Feather - C++**

The C++ code file runs on the ESP32 microcontroller and sets up a WiFi server to receive data from the Python code. The following is a summary of the main parts of the code:
1. The code starts by including several libraries, including the "Wire.h" library for I2C communication, the "WiFi.h" library for connecting to a WiFi network, the "Adafruit_PWMServoDriver.h" library for controlling servo motors with a PWM driver, and the "vector" library for storing data in arrays.
2. The code then defines some constants such as  `SERVOMIN`, `SERVOMAX` and `SERVO_FREQ`. These constants are used to set the minimum and maximum rotation for the servo motors and the PWM frequency.
3. The code then declares several functions:
  - `mapToServoRotation`: This function maps the coordinates of the face to the appropriate rotation for the servo motors.
  - `parseArray`: This function parses the data received from the Python code and converts it into an array of integers.
  - `blinkCycle`: every 5 seconds blinkCycle triggers a blinking of the eye by setting the eyelids from open to close to open with the servos
4. In the `setup()` function:
  - it starts the serial communication at 115200 baud rate.
  - it connects to the WiFi network by providing the SSID and password.
  - it starts the server and prints the IP address of the ESP32.
  - it initializes the PWM driver and sets the oscillator frequency and PWM frequency for the servo motors.
5. In the `loop()` function:
  - it checks for available clients and if a client is connected
  - it receives the data from the client, maps it to the servo rotations, sets the PWM values for the servo motors.
  - it calls the `blinkCycle`.
6. It also has a union byteToInt which is used to convert the binary data received from the client to int values.

**Python on PC**

The Python code file captures a webcam stream, uses OpenCV to detect a face, and sends the coordinates of the face to the ESP32 server. The following is a summary of the main parts of the code:
1. The code starts by importing several libraries, including `cv2 for OpenCV`, socket for network communication and struct for binary data.
2. The code then defines some constants such as `PORT_NUMBER` and `IP_ADDRESS`. These constants are used to set the port number and IP address for the server on the ESP32.
3. The code then loads the cascade classifier from a .xml file, this classifier is used to detect faces in the webcam stream.
4. The code then opens the webcam and connects to the server hosted on the ESP32.
5. While a connection stands, the following run in a loop:
  - it reads a frame from the webcam.
  - it converts the frame to grayscale.
  - it detects faces in the frame using the cascade classifier.
  - it finds the biggest face among all the faces detected.
  - it draws a rectangle around the biggest face.
  - it takes the center of the face and sends the x, y, w, h coordinates of the face to the server over a TCP connection.
6. It uses `struct.pack()` to pack the data into byte representation and sends it to the client via `client.sendall()`
7. Finally, it releases the webcam and closes the window and closes the connection to the server using `client.close()`

## Code segments
### Python script:
```python
import cv2
import socket
import struct
import time

PORT_NUMBER = 80
IP_ADDRESS = 'INSERT-IP_ADRESS'

# Load the cascade classifier
face_cascade = cv2.CascadeClassifier('/PATH_TO/haarcascade_frontalface_default.xml')

# Open the webcam
cap = cv2.VideoCapture(0)

# Connect to the server hosted on Arduino
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect((IP_ADDRESS, PORT_NUMBER))

while True:
    # Read a frame from the webcam
    ret, frame = cap.read()

    # Convert the frame to grayscale
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    # Detect faces in the frame
    faces = face_cascade.detectMultiScale(gray, 1.1, 4)

    # Draw a rectangle around the faces
    if len(faces) > 0:
        biggest_face = max(faces, key=lambda x: x[2]*x[3])
        x, y, w, h = biggest_face
        cv2.rectangle(frame, (x, y), (x+w, y+h), (255, 0, 0), 2)

        x += w/2
        y += h/2
        w, h, _ = frame.shape
        print("Face Coordinates:", x, y, w,h)

        face_coordinates = struct.pack('!iiii', int(x), int(y), int(w), int(h))

        # Send the face coordinates as an integer array to the server
        client.sendall(face_coordinates)

    # Display the frame
    cv2.imshow('Webcam', frame)

    # Exit the loop if the 'q' key is pressed
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# Release the webcam and close the window
cap.release()
cv2.destroyAllWindows()

# Close the connection to the server
client.close()
```

### C++ Script:
```cpp
#include <Wire.h>
#include <WiFi.h>
#include <Adafruit_PWMServoDriver.h>
#include <vector>

#define SERVOMIN 150
#define SERVOMAX 600
#define SERVO_FREQ 50

std::vector<int> mapToServoRotation(int x, int y, int w, int h);
std::vector<int> parseArray(WiFiClient client);
void blinkCycle(), recvOneChar(), showNewData();


const char* ssid = "NAME_HERE";
const char* password = "PASWORD_HERE";

int vert =  0;
int hor = 0;

int received;
boolean newData = false;
int turnTo = 90;
WiFiServer server(80);
Adafruit_PWMServoDriver pwm = Adafruit_PWMServoDriver();

void setup() {
 
  Serial.begin(115200);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  server.begin();
  Serial.println("Server started");
  Serial.print("Use this URL to connect: ");
  Serial.print("http://");
  Serial.print(WiFi.localIP());

  pwm.begin();
  pwm.setOscillatorFrequency(27000000);
  pwm.setPWMFreq(SERVO_FREQ);  // Analog servos run at ~50 Hz updates

  delay(10);
}

void loop() {
    WiFiClient client = server.available();
    if (client) {
        while (client.connected()) {
        if (client.available()) {
            std::vector<int> array = parseArray(client);
            int x = array[0];
            int y = array[1];
            int w = array[2];
            int h = array[3];

            std::vector<int> servo_rotations = mapToServoRotation(x, y, w, h);

            vert = map(servo_rotations[0], 0,180, SERVOMIN, SERVOMAX);
            hor = map(servo_rotations[1], 0,180, SERVOMIN, SERVOMAX);

            pwm.setPWM(0, 0, hor);
            pwm.setPWM(1, 0, vert);
            blinkCycle();


        }
        }
        client.stop();
    }
    blinkCycle();
}

// data type for converting byte to in array
union byteToInt {
    byte array[4];
    int val;
};

//reads from client and converts to int
std::vector<int> parseArray(WiFiClient client) {
    std::vector<int> array = {0, 0, 0, 0};
    
    byteToInt bti;

    for (int i = 0; i < 4; i++)
    {
        for (int j = 3; j >= 0; j--)
        {
            bti.array[j] = client.read();
        }

        array[i] = bti.val;
    }
    
    Serial.print("x,y,w,h = ");

    for (int i = 0; i < 4; i++)
    {
        Serial.print(array[i]);
        Serial.print(" ");
    }

    Serial.print("\n");
    client.flush();

    return array;
}

//takes face coordinate sand maps it to an appropriate servo rotation
// still needs some fine tuning and adjustments for camera angle and fov
std::vector<int> mapToServoRotation(int y, int x, int w, int h) {
    std::vector<int> servo_rotations = {0, 0};
    float x_ratio = (float)y / (float)w;
    float y_ratio = (float)x / (float)h;

    servo_rotations[0] = (int)map(x,0,h,108,54);
    servo_rotations[1] = (int)map(y,0,w,90,0);

    Serial.println("Servorot:");
    Serial.println(servo_rotations[0]);
    Serial.println(servo_rotations[1]);

    return servo_rotations;
}

//blinks every 5 seconds
void blinkCycle() {
    static unsigned long previousMillis = 0;
    unsigned long currentMillis = millis();
    if (currentMillis - previousMillis >= 5000) {

        int open = map(0, 0,180, SERVOMIN, SERVOMAX);
        int close = map(45, 0,180, SERVOMIN, SERVOMAX);
        
        pwm.setPWM(2, 0, open);
        pwm.setPWM(3, 0, close);
        delay(100);

        pwm.setPWM(2, 0, close);
        pwm.setPWM(3, 0, open);    
        delay(150);

        pwm.setPWM(2, 0, open);
        pwm.setPWM(3, 0, close);


        Serial.println("Blink");
        previousMillis = currentMillis;
    }
}
```
