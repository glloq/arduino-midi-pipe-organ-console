# arduino-midi-pipe-organ-console

i'm trying to help someone to update a console for a pipe organ  to midi with many arduino  
![schema principe](https://github.com/glloq/arduino-midi-pipe-organ-console/blob/main/console.png)

Each keyboard have one electric common (rail) and 54 inputs normaly open  
![schema principe](https://github.com/glloq/arduino-midi-pipe-organ-console/blob/main/keys.png)

Each Expression, Registers, Stopknobs and Pistons are also made the same ways with a commom cable and normaly open.

The pedals will use a simple rotary potentiometer to send the midi message


## Numbers of inputs 

- Keyboard 1 : 54 keys 
- Keyboard 2 : 54 keys 
- Pedal Keys  : 32 keys?
- Stopknobs : ?
- Expression : ?
- Combination Pistons : 39 
- Registers  : 42 

## MIDI Chanel to use

- Keyboard 1 on MIDI Channel 1
- Keyboard 2 on MIDI Channel 2
- Pedal Keys in MIDI. Channel 8
- Stopknobs on MIDI Channel 4
- Expression on MIDI Channel 7
- Combination Pistons on 6
- Registers on 5

## electronic choices

we'll build this around some arduino mega with some connection shield like this  
![schema principe](https://github.com/glloq/arduino-midi-pipe-organ-console/blob/main/shield%20mega.png)

The goal is to use one arduino MIDI usb compatible (leonardo or micro)  as a "Master" with many other arduino mega as "salve", each slave send the midi message to the master via I2C when there is a change on a input state.  
The master ask each slave if there is a new midi message, get the last one and send it via usb
Each slave has to memorise the news midi messages he need to send next, to avoid missing a new input state 


### Master code 

for arduino micro

```
#include <Wire.h>
#include <MIDIUSB.h> // to send midi message via USB

// I2C Slave Addresses for the Mega devices
#define MEGA1_ADDRESS 8 // keyboard 1 
#define MEGA2_ADDRESS 9 // keyboard 2
#define MEGA3_ADDRESS 10 // keyboard pedals and other?
#define MEGA3_ADDRESS 11 // other register, pistons expression pedal etc.. ?
// you can add other I2C adress if neededd

void setup() {
    // Initialize I2C as master
    Wire.begin();
    Serial.begin(115200);
}

void loop() {
    // Check for MIDI messages from each Mega
    checkMidiMessages(MEGA1_ADDRESS);
    checkMidiMessages(MEGA2_ADDRESS);
    checkMidiMessages(MEGA3_ADDRESS);
    checkMidiMessages(MEGA4_ADDRESS);
    delay(10);  // Small delay for stability
}

void checkMidiMessages(uint8_t slaveAddress) {
    Wire.beginTransmission(slaveAddress);
    Wire.write(0x00);  // Command to request MIDI message from the slave
    Wire.endTransmission();

    Wire.requestFrom(slaveAddress, 3); // Request 3 bytes (MIDI message)
    if (Wire.available() == 3) {
        uint8_t status = Wire.read();
        uint8_t data1 = Wire.read();
        uint8_t data2 = Wire.read();
        
        // Send the MIDI message via USB
        MidiUSB.sendMIDI(status, data1, data2);
        MidiUSB.flush();
    }
}

```

