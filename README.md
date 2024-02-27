// pc to mcu

#include <stdio.h>
#include <stdlib.h>
#include <windows.h>

#define COM_PORT "\\\\.\\COM4" 
#define BAUD_RATE 2400
#define BUFFER_SIZE 128

int main() {
    HANDLE hSerial;
    DCB dcbSerialParams = {0};
    COMMTIMEOUTS timeouts = {0};

    hSerial = CreateFileA(COM_PORT,
                        GENERIC_READ | GENERIC_WRITE,
                        0,
                        0,
                        OPEN_EXISTING,
                        FILE_ATTRIBUTE_NORMAL,
                        0);

    if (hSerial == INVALID_HANDLE_VALUE) {
        fprintf(stderr, "Error opening serial port\n");
        return 1;
    }
    dcbSerialParams.DCBlength = sizeof(dcbSerialParams);
    if (!GetCommState(hSerial, &dcbSerialParams)) {
        fprintf(stderr, "Error getting device state\n");
        CloseHandle(hSerial);
        return 1;
    }
    dcbSerialParams.BaudRate = BAUD_RATE;
    dcbSerialParams.ByteSize = 8;
    dcbSerialParams.StopBits = ONESTOPBIT;
    dcbSerialParams.Parity = NOPARITY;
    if (!SetCommState(hSerial, &dcbSerialParams)) {
        fprintf(stderr, "Error setting device parameters\n");
        CloseHandle(hSerial);
        return 1;
    }
    timeouts.ReadIntervalTimeout = 50;
    timeouts.ReadTotalTimeoutConstant = 50;
    timeouts.ReadTotalTimeoutMultiplier = 10;
    timeouts.WriteTotalTimeoutConstant = 50;
    timeouts.WriteTotalTimeoutMultiplier = 10;
    if (!SetCommTimeouts(hSerial, &timeouts)) {
        fprintf(stderr, "Error setting timeouts\n");
        CloseHandle(hSerial);
        return 1;
    }
    char buffer[BUFFER_SIZE];
    DWORD bytesRead;
    while (1) {
        if (!ReadFile(hSerial, buffer, BUFFER_SIZE, &bytesRead, NULL)) {
            fprintf(stderr, "Error reading from serial port\n");
            CloseHandle(hSerial);
            return 1;
        }
        if (bytesRead > 0) {
            for (DWORD i = 0; i < bytesRead; ++i) {
                putchar(buffer[i]);
            }
        }
    }
    CloseHandle(hSerial);
    return 0;
}


// mcu to pc


#include <EEPROM.h>
#define BAUD_RATE 2400

void setup() {
  Serial.begin(BAUD_RATE);
}

void loop() {
  while (Serial.available() > 0) {
    EEPROM.write(Serial.read(), Serial.read());
  }
  for (int i = 0; i < EEPROM.length(); ++i) {
    if (EEPROM.read(i) != 255) { // Check if data is written in the EEPROM location
      Serial.write(EEPROM.read(i));
      // Measure speed of transmission
      unsigned long start_time = millis();
      delay(1000); // Wait for 1 second
      unsigned long end_time = millis();
      unsigned long elapsed_time = end_time - start_time;
      unsigned long bits_transmitted = BAUD_RATE * elapsed_time / 1000;
      Serial.println(bits_transmitted);
    }
  }
}

