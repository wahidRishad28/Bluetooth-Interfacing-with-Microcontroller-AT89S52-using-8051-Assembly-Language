# Bluetooth-Interfacing-with-Microcontroller-AT89S52-using-8051-Assembly-Language
This project demonstrates wireless control and communication with an 8051 microcontroller using a Bluetooth (HC-05) module. It features multi-modal operation, including LED control, relay switching, Morse code translation, simple encryption, Morse decoding with LED visualization, and a basic calculator.


🛠 Key Features
Serial Communication at 9600 baud using Timer 1.

LED Control: Toggle 8 LEDs individually.

Relay Control: Safe switching with a transistor and protection diode.

Morse Code Translator: Display Morse equivalents of letters on an LCD.

Encryption Module: Simple Caesar cipher (+1 shift).

Morse Decoder: Real-time Morse code input via Bluetooth with LED feedback.

Basic Calculator: Supports addition, subtraction, and multiplication of two-digit numbers.

⚙️ Hardware Components
AT89S52 Microcontroller Board

HC-05 Bluetooth Module

16x2 LCD Display

Relay Module

LEDs, Resistors, Capacitors, Jumper Wires

🖥 Simulation
Circuit designed and verified using Proteus simulation software.

How it Works-
After initialization, the system prompts users via LCD to select a mode.

User commands are sent through Bluetooth, and the microcontroller processes them accordingly.

Real-time feedback is provided via the LCD, LEDs, or external devices connected through the relay.
