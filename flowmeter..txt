const int sensorPin=2;
volatile int pulseCount;
const int sensorInterrupt=0; //For Interrupt function we use pin 2 of arduino (interrupt pin 0 for this purpose)
float totalliquid=0;
float totalliquidprev=0;
float flow=0;
float flowLmin=0;
int oldTime=0;
int pinTono=7;
int margin=0; //ml
int baselineFLOW=0; // baseline value to compare with

//pushbuton
int inPin = 6;         // the button is connected here
int outPin = 8;       // the LED stating the Status goes to this pin
int state = HIGH;      // by default we set the ECO state
int reading;           // for reading the status
int previous = LOW;    // the previous reading
long time = 0;         // the last time the output pin was toggled
long debounce = 200;   // the debounce time for avoiding noise
//

void setup() {
  pinMode(inPin, INPUT);
  pinMode(outPin, OUTPUT);
  pinMode(sensorPin, INPUT);
  Serial.begin(9600);
  attachInterrupt(0, counter, RISING);
  pulseCount=0;

    // the for() loop saves some extra coding
  for (int pinNumber = 3; pinNumber < 7; pinNumber++) {
    if (pinNumber == inPin){
      continue;
    }
    pinMode(pinNumber, OUTPUT);
    digitalWrite(pinNumber, LOW);
     
  }
}

void loop() {

  amoroso();
  if(state == HIGH){
    margin = 250;
    baselineFLOW = 300;
  }
  else {
    margin=1000; 
    baselineFLOW=400;
  }
  int dif = (millis()-oldTime);
  int difV= (totalliquid-totalliquidprev);
  if( dif > 500){

    flow=(float)difV/(float)dif;
    flowLmin=flow*3600;
    
    Serial.print("ms: ");
    Serial.print(millis());
    Serial.print(". Pulse no.  ");
    Serial.print(pulseCount);
    Serial.print(". The Flow is:  ");
    Serial.print(flowLmin);
    Serial.print(" L/min");
    Serial.print(". Total Volume");
    Serial.print(totalliquid);
    Serial.println(" mL");
    oldTime = millis();
    totalliquidprev = totalliquid;
    }  

    // if the current expent water is lower than the baseline
  // turn off all LEDs
  if (totalliquid < baselineFLOW) {
    digitalWrite(5, HIGH);
    digitalWrite(4, LOW);
    digitalWrite(3, LOW);
  } // if the water consumption increases, turn a LED on
  else if (totalliquid >= baselineFLOW  && totalliquid < baselineFLOW+margin) {
    digitalWrite(5, LOW);
    digitalWrite(4, HIGH);
    digitalWrite(3, LOW);
      for (int i=30;i<400;i=i+50){
        tone(7,i,500);
        delay(100);
      }
     if(flow==0){
        exit(0);
      }
  } // if the temperature rises 4-6 degrees, turn a second LED on
  else if (totalliquid >= baselineFLOW + margin) {
    digitalWrite(5, LOW);
    digitalWrite(4, LOW);
    digitalWrite(3, HIGH);
    for (int i=30;i<2000;i=i+50){
      tone(7,i,100);
      delay(10);
     }
  if(flow==0){
    exit(0);
  }
  }
  
}

void counter()
{
   pulseCount++; //function to increment "count" by 1
   totalliquid=pulseCount*2.25;
}

void amoroso()
{
  reading = digitalRead(inPin);
  if (reading == HIGH && previous == LOW && millis() - time > debounce) {
    if (state == HIGH){
      state = LOW;
          }
    else
      state = HIGH;
    time = millis();    
  }
  digitalWrite(outPin, state);
  previous = reading;  
}
 
