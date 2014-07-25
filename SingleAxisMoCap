// Arduino Motion Control
// props to Frederic K. Macchi - author of this code


//----------------------
// Include LCD library
#include <LiquidCrystal.h>

//-----
// Pins 
const int pinCAM = 13;        // Camera release relay 
const int pinSTOP = 12;       // Instant stop button 

const int pinSTEP_SLP = 11;   // Sleep on Easydriver
const int pinSTEP_MS = 10;    // Microsteps on Easydriver
const int pinSTEP_DIR = 9;    // Direction on Easydriver
const int pinSTEP_ST = 8;     // Step on Easydriver

const int pinLCD_RS = 7;      // RS on LCD
const int pinLCD_EN = 6;      // Enable on LCD
const int pinLCD_D4 = 5;      // D4 on LCD
const int pinLCD_D5 = 4;      // D5 on LCD
const int pinLCD_D6 = 3;      // D6 on LCD
const int pinLCD_D7 = 2;      // D7 on LCD

const int pinPOTI = A0;       // Rotary knob
const int pinBTN_S = A1;      // Start button
const int pinBTN_3 = A2;      // Button 3
const int pinBTN_2 = A3;      // Button 2
const int pinBTN_1 = A4;      // Button 1

//----------
// LCD setup
LiquidCrystal LCD(pinLCD_RS, pinLCD_EN, pinLCD_D4, pinLCD_D5, pinLCD_D6, pinLCD_D7);

//----------
// Variables
int numPotiVal = 0;		// Default value
int numStepsDone = 0;	// Default value

// Put exposure time here
// Example: 5 seconds exposure -> numReleaseTime = 5000
// I recommend adding a second rotary knob for this value so you 
// don't have to upload the sketch every time the exposure time changes
int numReleaseTime = 1000;  // Seconds *1000

boolean boolDIR = false; 	// Forward?
boolean boolREL = false; 	// Release?
boolean boolNX = false;  	// Repeat steps?
boolean boolRUN = false;	// Is the start button pressed?
boolean boolACTIVE = false;	// Is there an active run?

//------
// Setup
void setup(){
  
  //Serial.begin(9600);
  
  // Setup pins
  pinMode(pinBTN_S, INPUT);
  pinMode(pinBTN_1, INPUT);
  pinMode(pinBTN_2, INPUT);
  pinMode(pinBTN_3, INPUT);
  pinMode(pinPOTI, INPUT);
  
  pinMode(pinSTOP, INPUT);
  pinMode(pinCAM, OUTPUT);
  
  pinMode(pinSTEP_ST, OUTPUT);
  pinMode(pinSTEP_DIR, OUTPUT);
  pinMode(pinSTEP_MS, OUTPUT);
  pinMode(pinSTEP_SLP, OUTPUT);
  
  // Write pins
  digitalWrite(pinCAM, LOW);
  digitalWrite(pinSTEP_ST, LOW);
  digitalWrite(pinSTEP_MS, LOW);
  digitalWrite(pinSTEP_SLP, LOW);
  
  // Startup message on LCD
  LCD.begin(16, 2);
  LCD.print(" MOTION CONTROL");
  LCD.setCursor(0, 1);
  LCD.print("   FKM1900.DE");
  
  delay(3000); // Show message for 3 seconds
  LCD.clear();
}

//-----
// Loop
void loop(){
  
  // Update display with 25 fps
  if(millis()%40 == 0){ 
    
    // Update values
	// Values are read as analog input due to a different approach in the beginning. 
	// It should work as digital input as well. If you do so, change the pin setup.
    numPotiVal = map(analogRead(pinPOTI), 0, 1023, 0 , 100); // Map poti input 0-1023 to output 0-100
    if(analogRead(pinBTN_1) >= 512){ boolDIR = true; } else { boolDIR = false; }
    if(analogRead(pinBTN_2) >= 512){ boolREL = true; } else { boolREL = false; }
    if(analogRead(pinBTN_3) >= 512){ boolNX = true; } else { boolNX = false; }
    if(analogRead(pinBTN_S) >= 512){ boolRUN = true; } else { boolRUN = false; boolACTIVE = false; digitalWrite(pinSTEP_SLP, LOW); }
  
    // Update display
    UpdateDisplay(); 
    
    // Do steps
    if(!boolACTIVE && boolRUN){ DoStep(numPotiVal); }
  }
}

//--------------
// UpdateDisplay
void UpdateDisplay(){
  
  // First row
  String strBtnState = "";
  if(boolDIR){ strBtnState = String(strBtnState+"-> "); } else { strBtnState = String(strBtnState+"<- "); }
  if(boolREL){ strBtnState = String(strBtnState+"REL "); } else { strBtnState = String(strBtnState+"--- "); }
  if(boolNX){ strBtnState = String(strBtnState+"NX "); } else { strBtnState = String(strBtnState+"1X "); }
  if(boolRUN){ strBtnState = String(strBtnState+"RUN"); } else { strBtnState = String(strBtnState+"STOP"); }
  
  // Fill line end with blanks
  if(strBtnState.length() < 16){
    for(int i = 0; i <= 16 -strBtnState.length(); i++){
      strBtnState = String(strBtnState+" ");
    }
  }
  LCD.setCursor(0, 0); // Set display cursor to first line
  LCD.print(strBtnState);

  // Second row
  String strPotiState = String("DIST/STEP: "+String(numPotiVal)+"MM");
  
  // Fill line end with blanks
  if(strPotiState.length() < 16){
    for(int j = 0; j <= 16 -strPotiState.length(); j++){
      strPotiState = String(strPotiState+" ");
    }
  }
  LCD.setCursor(0, 1); // Set display cursor to second line
  LCD.print(strPotiState);
}

//-------
// RepeatDoStep
void RepeatDoStep(int numMM){
  delay(500);
  DoStep(numMM);
}

//-------
// DoStep
void DoStep(int numMM){
  boolACTIVE = true;
  numStepsDone = 0;
  
  // Setup stepper motor
  digitalWrite(pinSTEP_MS, LOW);
  if(boolDIR){ digitalWrite(pinSTEP_DIR, HIGH); } else { digitalWrite(pinSTEP_DIR, LOW); }
  digitalWrite(pinSTEP_SLP, HIGH);

  // Do steps
  // My stepper motor does 200 steps per revolution. 
  // The threaded spindle has a flank lead of 4 mm. This means 1/4 revolution lets the flange nut advance 1mm.
  // Therefore, 1/4*200 = 50 steps are necessary for 1 mm horizontal movement of the platform.
  // If your stepper motor does more/less steps per revolution you will have to adjust the multiplier.
  for(int i=1; i <= 50*numMM; i++){
    
    // Check inputs
    if(numStepsDone <= 50*numMM && digitalRead(pinSTOP) == HIGH && analogRead(pinBTN_S) >= 512){
      digitalWrite(pinSTEP_ST, HIGH);
      delayMicroseconds(200);
      digitalWrite(pinSTEP_ST, LOW);
      delayMicroseconds(10000);
      numStepsDone++;
      
      // No more steps to do
      if (numStepsDone == 50*numMM){
        
        // Release if REL is true
        if(boolREL){
          delay(500);
          digitalWrite(pinCAM, HIGH);
          delay(500);
          digitalWrite(pinCAM, LOW);
          delay(numReleaseTime+500);
        }
        
        // Repeat steps if NX is true
        if (boolNX){
          digitalWrite(pinSTEP_SLP, LOW);
          RepeatDoStep(numMM); 
          return; 
        }
      }
    } else {
      
      // Move 1 mm back if one of the instant stop buttons has been pressed
      if(digitalRead(pinSTOP) == LOW){
        // Reverse direction
        if(!boolDIR){ digitalWrite(pinSTEP_DIR, HIGH); } else { digitalWrite(pinSTEP_DIR, LOW); }
        for(int j=1; j <= 50; j++){
            digitalWrite(pinSTEP_ST, HIGH);
            delayMicroseconds(200);
            digitalWrite(pinSTEP_ST, LOW);
            delayMicroseconds(10000);
        }
      }

      // Turn motor off and return
      digitalWrite(pinSTEP_SLP, LOW);
      return;
    }
  }
}

