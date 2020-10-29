# Record sensor data to google sheet using NodeMCU Record temperature and humidity value DHT11
- ESP 8266
- DHT11
## Step 1: Create a spreadsheet and prepare it to receive data from a device using GScript
  - Create a sheet
  - Name the document title (Data Sensor) and name "SHEET" (temphum)
  - TOOL -> Script editor -> 1 tab editor (rename script - temphum_log)
  - copy code and past và Gscript
## Flowing example
 
// Script examples
// https://developers.google.com/adwords/scripts/docs/examples/spreadsheetapp

var SS = SpreadsheetApp.openById('1kUNeFjmRXzkU3YXHYwKDSCfbr7b50QPtXiaK8x3oA4M');    //Enter Your Sheet ID Got From Sheet URL Link
var sheet = SS.getSheetByName('temphum');      // Enter your sheet name here, In my case it is TempSheet
var str = "";

function onOpen(){
  var ui = SpreadsheetApp.getUi();
  ui.createMenu('ESP8266_Temp_Logger')
  .addItem('Clear', 'Clear')
  .addToUi();
}

function Clear(){
  sheet.deleteRows(4, sheet.getLastRow());
  SS.toast('Chart cleared', 'ESP8266_Temp_Logger', 5);
}

function doPost(e) {

  var parsedData;
  var result = {};
  
  try { 
    parsedData = JSON.parse(e.postData.contents);
  } 
  catch(f){
    return ContentService.createTextOutput("Error in parsing request body: " + f.message);
  }
   
  if (parsedData !== undefined){
    // Common items first
    // data format: 0 = display value(literal), 1 = object value
    var flag = parsedData.format;
    
    if (flag === undefined){
      flag = 0;
    }
    
    switch (parsedData.command) {
      case "appendRow":
         var tmp = SS.getSheetByName(parsedData.sheet_name);
         var nextFreeRow = tmp.getLastRow() + 1;
         var dataArr = parsedData.values.split(",");
         
         tmp.appendRow(dataArr);
         
         str = "Success";
         SpreadsheetApp.flush();
         break;     
       
       
    }
    
    return ContentService.createTextOutput(str);
  } // endif (parsedData !== undefined)
  
  else{
    return ContentService.createTextOutput("Error! Request body empty or in incorrect format.");
  }
  
  
}


function doGet(e){
  
  var val = e.parameter.value;
  var cal = e.parameter.cal;
  var read = e.parameter.read;
  
  if (cal !== undefined){
    return ContentService.createTextOutput(GetEventsOneWeek());
  }
  
  if (read !== undefined){
    var now = Utilities.formatDate(new Date(), "EST", "yyyy-MM-dd'T'hh:mm a'Z'").slice(11,19);
    //sheet.getRange('D1').setValue(now);
    //var count = (sheet.getRange('C1').getValue()) + 1;
    sheet.getRange('C1').setValue(count);
    return ContentService.createTextOutput(sheet.getRange('A1').getValue());
  }
  
  if (e.parameter.value === undefined)
    return ContentService.createTextOutput("No value passed as argument to script Url.");
    
  var range = sheet.getRange('A1');
  var retval = range.setValue(val).getValue();
  var now = Utilities.formatDate(new Date(), "EST", "yyyy-MM-dd'T'hh:mm a'Z'").slice(11,19);
//  sheet.getRange('B1').setValue(now);
  sheet.getRange('B1').setValue("Humidity");
  //sheet.getRange('C1').setValue('0');
  
  if (retval == e.parameter.value)
    return ContentService.createTextOutput("Successfully wrote: " + e.parameter.value + "\ninto spreadsheet.");
  else
    return ContentService.createTextOutput("Unable to write into spreadsheet.\nCheck authentication and make sure the cursor is not on cell 'A1'." + retval + ' ' + e.parameter.value);
}

function GetEventsOneWeek(){
  var Cal = CalendarApp.getCalendarsByName('Test REST API')[0];
  // Need to create 2 separate Date() objects. Cannot do 'OneWeekFromNow = Nowjs' to 
  // simply get it's value and use that later without modifying 'Now'
  // since in JS, an object is automatically passed by reference
  var Now = new Date();
  var OneWeekFromNow = new Date();
  OneWeekFromNow.setDate(Now.getDate() + 7);
  //Logger.log(Now);
  //Logger.log(OneWeekFromNow);
  var events = Cal.getEvents(Now, OneWeekFromNow);
  //Logger.log(events.length);
  var str = '\nEvent Title,\tDescription,\tRecurring?,\tAll-day?,\tFirst Reminder (in minutes before event)\n';
  for (var i = 0; i < events.length; i++){
    str += events[i].getTitle() + ',\t' + events[i].getDescription() + ',\t' + events[i].isRecurringEvent() +  ',\t' + events[i].isAllDayEvent() + ',\t' + events[i].getPopupReminders()[0];
    str += '\n';
  }
  //Logger.log(str);
  return str;
}
## =========================
Edit SHEET NAME and SHEET ID in the code
SHEET NAME: temphum
SHEET ID : 
  - tab gg sheet link URL: https://docs.google.com/spreadsheets/d/1kUNeFjmRXzkU3YXHYwKDSCfbr7b50QPtXiaK8x3oA4M/edit#gid=0
  - ID: 1kUNeFjmRXzkU3YXHYwKDSCfbr7b50QPtXiaK8x3oA4M
==>> Save file -. Tab script Publish -> Deploy as Web app..
    - project name
    - Who has as tho app: Anyone, even anonymous
    ===> Deloy _. next acep...
 *** Copy ID Cript: 
 Link: https://script.google.com/macros/s/AKfycbxwoQGvDkf6tMQTM1wrA5WckBtt6sWy_cgycc0gpD6gvPoxHM_3/exec
 ID: AKfycbxwoQGvDkf6tMQTM1wrA5WckBtt6sWy_cgycc0gpD6gvPoxHM_3
## =========================
4 Main library:
  - ESP8266WWiFi
  - HTTPSRedirect: easily communicate with the server via http and https
  - DebugMacros: 
  - DHT
## =========================
#include <ESP8266WiFi.h>
#include "HTTPSRedirect.h"
#include "DebugMacros.h"
#include <DHT.h>
#define DHTPIN D4 // Ghim mà DHT được kết nối
#define DHTTYPE DHT11 // Khai báo Loại DHT
DHT dht ( DHTPIN, DHTTYPE ) ;
## =========================
float h;
float t;
String sheetHumid = "";
String sheetTemp = "";
const char* ssid = " ";                //Put wifi ssid within the quotes
const char* password = " ";         //Put WiFi password within the quotes
## =========================
const char* host = "script.google.com";
const char *GScriptId = "enter your GSCript ID here"; // Replace with your own google script id
const int httpsPort = 443; //the https port is same

