# SCC-task

#define encoderpin1 2
#define encoderpin2 3
#define driverpin1 4
#define driverpin2 5
#define pwmPin 6
#define motorpin1 8
#define motorpin2 7
#define pwmPin1 10
#define ledPin 13
volatile int count =0;//initial count

float target=1200 ;
//set target position
int prevpos=0;
float prevtime=0;
int eprev =0;
float eintegral =0;



void setup()
{
  

  
  
  pinMode(encoderpin1, INPUT_PULLUP);  
  pinMode(encoderpin2, INPUT_PULLUP);
  
  
  //calling the interrupt function
  attachInterrupt(digitalPinToInterrupt(encoderpin1), Encodercount, RISING);


  pinMode(driverpin1, OUTPUT);//control the motor1
  pinMode(driverpin2, OUTPUT);
  pinMode(pwmPin,OUTPUT);//enable pin 2and 3
  
  
  
  pinMode(motorpin1, OUTPUT);//control motor 2
  pinMode(motorpin2, OUTPUT);  
  pinMode(pwmPin1,OUTPUT);//enable pin 1 and 2
  

  
  pinMode(ledPin, OUTPUT);
  
  Serial.begin(9600);
  
  
  
 
}




void loop()
{
  
  
  
  float kp=1,kd=0.025,ki=0;//pid constants

  float currtime = micros();//record the time
  
  float deltat = (currtime-prevtime)/1.0e6;//change in time 
  
  
  prevtime=currtime;//save the current time 
  
  
  int e = count - target;//the error

  
  float dedt = (e - eprev)/deltat;//derivative
  
   eintegral = eintegral + e*deltat;//integral 
  
  
  float u = kp*e + kd * dedt + ki*eintegral;//pdi control 
  
  eprev = e;
  
  //using the pwm switch control the position of the motor
  
  float pwr=fabs(u);//the power delivered to motor
  
  if(pwr>255)//if the u value exceeds 255
   {
    
       pwr=255;
  
   }
  
  
  int dir =1;//set direction
  
  
  if(u<0)//if the u value is negative
  {
    dir=-1;

  }
  
 
  setMotor(dir,pwr,pwmPin,driverpin1,driverpin2);//for the encoder and motor1
  setMotor(dir,pwr,pwmPin1,motorpin1,motorpin2);//for the motor2
   
 
  Serial.print("Target: ");//print target
  Serial.println(target);
  
  
  Serial.print("Count: ");//print count
  Serial.println(count);
  
  
  
  if(abs(u)<5)//blink the led when motor is about to stop
  {
    
    digitalWrite(ledPin,HIGH);
    
    delay(100);
    
    digitalWrite(ledPin,LOW);
    
    delay(100);
    
    
  }
  else
  {
    
    digitalWrite(ledPin,LOW);
    
    
  }  
  
  


}



void setMotor(int dir, int pwmVal, int pwm, int dr1, int dr2)
{
  
  analogWrite(pwm,pwmVal);

  if(dir==1)//clockwise rotation of the motor
  {
    
  digitalWrite(dr1,LOW);
  digitalWrite(dr2,HIGH);
  

  

  }
  else if(dir==-1)//motor rotates anticlockwise
    
  {
    
   digitalWrite(dr1,HIGH);
   digitalWrite(dr2,LOW);
  
    
    
    
  }
  
  
  else//motor stops
    
  {
    
    
    
    
     digitalWrite(dr1,LOW);
     digitalWrite(dr2,LOW);
    

  }  
