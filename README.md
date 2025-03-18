# arduino-midi-pipe-organ-console

i'm trying to help someone to update a console for a pipe organ to midi with many arduino  
![schema principe](https://github.com/glloq/arduino-midi-pipe-organ-console/blob/main/console.png)

Each keyboard have one electric common (rail) and 54 inputs normaly open  
![schema principe](https://github.com/glloq/arduino-midi-pipe-organ-console/blob/main/keys.png)

Each Expression, Registers, Stopknobs and Pistons are also made the same ways with a commom cable and normaly open.

The pedals will use a simple rotary potentiometer to send the midi message


## Numbers of inputs 

It's probably easyer to wire each blocs on one arduino (we can use a arduino mega foe each keyboard and one or two for the contacts fixed on the console)
![schema principe](https://github.com/glloq/arduino-midi-pipe-organ-console/blob/main/console%20repartition%201.png)

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

The master ask each slave if there is a new midi message, if there is a new messages he get the last one and send it via usb.  
Each slave has to memorise the news midi messages he need to send next, to avoid missing a new input state (we'll use a fifo buffer) 


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

### Slave code 

we will need to have adaptive code to allow associating an input with a type of message to send.  
In case of keyboard notes we will change the byte to indicate a On or Off note   
In case of a Program change, we will change the byte to indicate a On or Off  

for each input we'll need to remember the channel and the type of the message to send (how to react to a input change)  

I made a code that use a structure to stores all the information that the Arduino must send when a state change

```
typedef struct {
  byte midiChannel;     // MIDI Canal  (0-15)
  int pin;              // Pin 
  byte messageType;     // MIDI message Type (0 for a Note, 1 for a Program Change)
  byte param1;          // Parameter 1 (MIDI number or used to send a On PC)
  byte param2;          // Parameter 2 (only used to send a Off PC)
} MidiInput;

```
each input state will be stored in a table to know if there is a change in state (to send only once the midi message) and to know witch message to send (with a keys, we'll send a noteOn when closed and noteOff when open) 


i'll need to add a messageType for the potentiometer, or another midi message we'll need for the pipe organ.


