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




