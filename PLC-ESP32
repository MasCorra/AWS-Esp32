/*
Specify its number into DBNum variable

-Both small and large data transfer are performed (see DO_IT_SMALL)
-During the loop, try to disconnect the ethernet cable.
  The system will report the error and will reconnect automatically
  when you re-plug the cable.
-For safety, this demo *doesn't write" data into the PLC, try
  yourself to change ReadArea with WriteArea.
-This demo uses ConnectTo() with Rack=0 and Slot=2 (S7300)
  -If you want to connect to S71200/S71500 change them to Rack=0, Slot=0.
  -If you want to connect to S7400 see your  hardware configuration,
  -If you want to work with  a LOGO 0BA7 or S7200 please refer to the
  documentation and change
  Client.ConnectTo(<IP>, <Rack>, <Slot>);
  with the couple
  Client.SetConnectionParams(<IP>, <LocalTSAP>, <Remote TSAP>);
  Client.Connect();

NodeMCU 1.0 ESP-12E ESP8266 supported
*/

#define S7WIFI

#include "WiFi.h"
#include "Platform.h"
#include "Settimino.h"

//Uncomment next line to perform small and fast access
#define DO_IT_SMALL

//Enter a MAC address and IP for your controller below.
// The IP adress will be dependent on your local network
byte mac[]={ 0x90, 0xA2, 0xDA, 0x0F, 0x08, 0xE1 };

IPAddress Local(10, 10, 0, 2); //Local Address
IPAddress PLCip(192, 168, 0, 124); //PLC Address

// Following constants are needed if you are connecting via WIFI
// The ssid is the name of my WIFI network (the password obviously is wrong)
char ssid[] = "Wokwi-GUEST";    // Your network SSID (name)
char pass[] = "";  // Your network password (if any)
IPAddress Gateway(10, 0, 0, 1);
IPAddress Subnet(255, 255, 0, 0);

int DBNum = 1; //This DB must be present in your PLC
byte Buffer(1024);

//S7Client will create a EthernetClient as TCP Client
S7Client Client;


unsigned long Elapsed; //To calc the execution time

//------------------------------------------------------------------
// Setup: Init Ethernet and Serial port
//------------------------------------------------------------------
void setup() {
  //Open serial communications and wait for port to open;
  Serial.begin(115200);
  while (!Serial) {
    ; //wait for serial to port to connect. Needed for Leonardo only
  }

  
//-----------------------Network Initialization

#ifdef S7WIFI
//--------------------------------------------- ESP8266 Initialization    
    Serial.println();
    Serial.print("Connecting to ");
    Serial.println(ssid);
    WiFi.begin(ssid, pass);
    WiFi.config(Local, Gateway, Subnet);
    while (WiFi.status() != WL_CONNECTED) 
    {
        delay(500);
        Serial.print(".");
        //Serial.print(WiFi.status());
    }
    Serial.println("");
    Serial.println("WiFi connected");  
    Serial.print("Local IP address : ");
    Serial.println(WiFi.localIP());

    // Print subnet mask
    Serial.print("Subnet mask: ");
    Serial.println(WiFi.subnetMask());

    // Print gateway IP
    Serial.print("Gateway IP: ");
    Serial.println(WiFi.gatewayIP());
#else
//--------------------------------Wired Ethernet Shield Initialization    
    // Start the Ethernet Library
    EthernetInit(mac, Local);
    // Setup Time, someone said me to leave 2000 because some 
    // rubbish compatible boards are a bit deaf.
    delay(2000); 
    Serial.println("");
    Serial.println("Cable connected");  
    Serial.print("Local IP address : ");
    Serial.println(Ethernet.localIP());
#endif   
}

//------------------------------------------------------------------
//Connects to the PLC
//------------------------------------------------------------------
/*
bool Connect ()
{
    int Result=ConnectTo(PLCip,
                                  0,  //Rack (see the doc.)
                                  0); //Slot (see the doc.) per S7-1200 (0,0) o (0,1)
    Serial.print("Connecting to ");Serial.println(PLCip);
    if (Result==0)
    {
      Serial.print("Connected ! PDU Length = ");Serial.println(Client.GetPDULength());
    }
    else
      Serial.println ("Connection error");
    return Result==0;
}
*/

bool ConnectToPLC() {
  Serial.print("pippo");
  int result = Client.ConnectTo(PLCip, 0, 0); // Replace 0 and 2 with your Rack and Slot numbers
  Serial.print("Connecting to ");Serial.println(PLCip);
  
  if (result==0)
    {
      Serial.print("Connected ! PDU Length = ");Serial.println(Client.GetPDULength());
    }
  else
      Serial.println ("Connection error");
  return result==0;
}

//------------------------------------------------------------------
//Dumps a buffer (a very rough routine)
//------------------------------------------------------------------
void Dump (void *Buffer, int Length)
{
  int i, cnt=0;
  pbyte buf;

  if(Buffer!=NULL)
    buf = pbyte(Buffer);
  else
    buf = pbyte(&PDU.DATA[0]);

  Serial.print("[ Dumping ");Serial.print(Length);
  Serial.println(" bytes ]=========================");
  for (i=0;i<Length;i++)
  {
    cnt++;
    if (buf[i]<0x10)
      Serial.print("0");
    Serial.print(buf[i], HEX);
    Serial.print(" ");
    if(cnt==16)
    {
      cnt=0;
      Serial.println();
    }
  }
  Serial.println("=======================");
}
//------------------------------------------------------------------
//Prints the Error number
//------------------------------------------------------------------
void CheckError (int ErrNo)
{
  Serial.print("Error No. 0x");
  Serial.println(ErrNo, HEX);

  //Check if it's a Server Error => we need to disconnect
  if (ErrNo & 0x00FF)
  {
    Serial.println("SEVERE ERROR, disconnecting");
    Client.Disconnect();
  }
}
//------------------------------------------------------------------
//Profiling routines
//------------------------------------------------------------------
void MarkTime()
{
  Elapsed=millis();
}
//------------------------------------------------------------------
void ShowTime()
{
  //Calcs the time
  Elapsed=millis()-Elapsed;
  Serial.print("Job time (ms) :");
  Serial.println(Elapsed);
}
//------------------------------------------------------------------
//Main Loop
//------------------------------------------------------------------
void loop() {
  int Size, Result;
  void *Target;
  Serial.print("1");
#ifdef DO_IT_SMALL
  Serial.print("2");
  Size=10;
  Target = NULL;     //Uses the internal Buffer (PDU.DATA[])
#else
  Serial.print("3");
  Size=1024;
  Target = &Buffer;  //Uses a larger buffer
#endif

  //Connection
  while (!Client.Connected)
  {Serial.print("4");
    if (!ConnectToPLC())
    
      delay(500);
  }
  
  Serial.print ("Reading ");Serial.print(Size);Serial.print(" bytes from DB");Serial.println(DBNum);
  //Get the current tick
  MarkTime();
  Serial.print("4");
  Result=Client.ReadArea(S7AreaDB, //We are requesting DB access
                        DBNum,     //DB Number
                        0,         //Start from byte N.0
                        Size,      //We need "Size" bytes
                        Target);   //Put them into our target (Buffer or PDU)
  Serial.print("5");
  if (Result==0)
  {
    ShowTime();
    Dump(Target, Size);
  }
  else
    CheckError(Result);

  delay(500);
}
