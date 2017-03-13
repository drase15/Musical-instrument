# Musical-instrument

My musical instrument uses 2 cap sensors, made from conductive paint on two pieces of wood. One controls the frequency of a sine curve in Processing, the other controls the amp of the sound produced from the sine curve. In the future I hope to be able to have the sensors control the gain of two different sound bites in processing, but for now I am happy with the result of the musical instrument. The use of the 10M ohm resistors make it so one does not have to directly touch the paint in order to change the values sent from Arduino. 

![alt text](http://i.imgur.com/khDw1K1.jpg)
![alt text](http://i.imgur.com/SNwW20U.jpg)
![alt text](http://i.imgur.com/Z2qAaz0.jpg?1)
![alt text](http://i.imgur.com/pjWeewf.jpg)
![alt text](http://i.imgur.com/EgoIhr2.jpg)





Arduino Code for Capsense

include <CapacitiveSensor.h>

/*
 * CapitiveSense Library Demo Sketch
 * Paul Badger 2008
 * Uses a high value resistor e.g. 10M between send pin and receive pin
 * Resistor effects sensitivity, experiment with values, 50K - 50M. Larger resistor values yield larger sensor values.
 * Receive pin is the sensor pin - try different amounts of foil/metal on this pin
 */


CapacitiveSensor   cs_4_2 = CapacitiveSensor(4,2);        // 10M resistor between pins 4 & 2, pin 2 is sensor pin, add a wire and or foil if desired
CapacitiveSensor   cs_8_6 = CapacitiveSensor(8,6);        // 10M resistor between pins 8 & 6, pin 6 is sensor pin, add a wire and or foil
//CapacitiveSensor   cs_4_8 = CapacitiveSensor(4,8);        // 10M resistor between pins 4 & 8, pin 8 is sensor pin, add a wire and or foil

void setup()                    
{
   cs_4_2.set_CS_AutocaL_Millis(0xFFFFFFFF);     // turn off autocalibrate on channel 1 - just as an example
   Serial.begin(9600);
}

void loop()                    
{
    long start = millis();
    long total1 =  cs_4_2.capacitiveSensor(30);
    long total2 =  cs_8_6.capacitiveSensor(30);
    //long total3 =  cs_4_8.capacitiveSensor(30);

    //Serial.print(millis() - start);        // check on performance in milliseconds
    //Serial.print("\t");                    // tab character for debug windown spacing

    Serial.print(total1);                  // print sensor output 1
    Serial.print("\t");
    Serial.println(total2);                  // print sensor output 2
    //Serial.print("\t");
    //Serial.println(total3);                // print sensor output 3

    delay(10);                             // arbitrary delay to limit data to serial port 
}







Processing Code

/*
Read a pair of numbers separated by a comma from the serial port, presumably sent by an Arduino
Use the first number to control the amplitude and the second number the frequency of a sine wave
*/

import processing.serial.*;    // Serial port for communication with Arduino
import ddf.minim.*;            // Minim library for generating sounds
import ddf.minim.signals.*;  // Minim library that provides sine wave

Serial myPort;        // The serial port
float freq;           // sine wave frequency
float amp;            // sine wave amplitude

Minim       minim;    // a Minim object
AudioOutput out;      // our audio output
SineWave sine;        // a function to generate the values of a sine wave

void setup () {
  // Open whatever port is the one you're using.
  // DON'T FORGET TO CHANGE THIS FOR YOUR COMPUTER
  myPort = new Serial(this, Serial.list()[2], 9600);

  // don't generate a serialEvent() unless you get a newline character:
  myPort.bufferUntil('\n');

  minim = new Minim(this); // Instantiate the minim object

  // Connect the output of Minim to the audio output 
  // of my laptop
  out = minim.getLineOut(Minim.STEREO);  

  // create a sine wave oscillator
  // Frequency = 440 Hz
  // Amplitude = 0.5
  // Use the same sample rate as my output  
  sine = new SineWave(440, 0.5, out.sampleRate()); 

  //connect the sine wave generator to my audio output
  out.addSignal(sine);
}
void draw () {
  sine.setFreq( freq );
  sine.setAmp( amp );

 // println(freq + ',' + amp);
}

void serialEvent (Serial myPort) {
  // get the ASCII string:
  String inString = myPort.readStringUntil('\n');

  if (inString != null) {
    // println(inString);
    // trim off any whitespace:
    inString = trim(inString);
    // split the input string at the commas
    // and convert the sections into integers:
    int sensors[] = int(split(inString, '\t'));

    // if we have received all the sensor values, use them:
    if (sensors.length == 2) {
      amp = map(sensors[1], 0, 1023, 0, 1);
      freq = map(sensors[0], 0, 1023, 110, 880);
    }
  }
}
