```javascript
//push new events to calendar
function pushToCalendar() {

  //spreadsheet variables
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName("SHEET NAME")
  var firstRow = 2;
  var nRows = sheet.getLastRow();
  var firstCol = 1;
  var nCols = 15; //remember 0 indexed
  var eventLocationCol = 2;
  var eventDescCol = 8;
  var eventNameCol = 9;
  var eventTimeStartCol = 10;
  var eventTimeEndCol = 11;
  var pushDoneCol = 12;
  var eventIDCol = 13;
  var lastActionCol = 14;
  var updateDateCol = 15;

  var range = sheet.getRange(firstRow,firstCol,nRows,nCols); //%** num columns must be updated if we modify the sheet columns
  var values = range.getValues();
//  var updateRange = sheet.getRange('T1'); //%**


  //calendar variables
  var calendar = CalendarApp.getCalendarById('4upcttql0tbavhog1d4s8qd4qs@group.calendar.google.com')

  //current date, time and timezone

  //var dateToday = Utilities.formatDate(new Date(), "GMT+1", "dd/MM/yyyy");

  //https://stackoverflow.com/questions/18596933/google-apps-script-formatdate-using-users-time-zone-instead-of-gmt
  //var addedDate = sheet.getRange(1,1).getValue();
  var dateToday = Utilities.formatDate(new Date(), SpreadsheetApp.getActive().getSpreadsheetTimeZone(), "dd/MM/yyyy hh:mm");
  //Utilities.formatDate(date, timeZone, format)

//show updating message
// updateRange.setFontColor('red')

  var numValues = 0;
  for (var i = 0; i < values.length; i++) {
    //check to see if event name, start and end date are filled
    //if ((values[i][0].length > 0) && (values[i][1] > values[i][2] )) {
      if (values[i][eventNameCol].length > 0) { //eventname

        //create event https://developers.google.com/apps-script/class_calendarapp#createEvent
        var newEventTitle = values[i][eventNameCol];
        var newEventDesc = values[i][eventDescCol];
        var newEventStart = values[i][eventTimeStartCol];
        var newEventEnd = values[i][eventTimeEndCol];
        var newEventLocation = values[i][eventLocationCol];

        //check if event ID exists
        //if (values[i][8] != '-Done-') {
        if (values[i][eventIDCol].length < 1) {     // if no entry

          // setting description: https://stackoverflow.com/questions/31548227/google-apps-script-setting-color-of-an-event-using-calendar-api
          var newEvent = calendar.createEvent(newEventTitle, newEventStart, newEventEnd,{location:newEventLocation, description:newEventDesc});

          //get ID
          var newEventId = newEvent.getId();

          //mark as entered, enter ID and date
          sheet.getRange(i+2,pushDoneCol+1).setValue('-Done-');
          sheet.getRange(i+2,pushDoneCol+1).setFontColor('red');

          sheet.getRange(i+2,lastActionCol+1).setValue('-New-');
          sheet.getRange(i+2,lastActionCol+1).setFontColor('red');

          sheet.getRange(i+2,eventIDCol+1).setValue(newEventId);
          sheet.getRange(i+2,updateDateCol+1).setValue(dateToday);
          sheet.getRange(i+2,updateDateCol+1).setFontColor('black');

        }
        else { //pull info from existing event

          var oldEventID = values[i][eventIDCol];

          //https://developers.google.com/apps-script/reference/calendar/calendar-app#getEventSeriesById(String)

          var oldEvent = calendar.getEventById(oldEventID);

          var oldEventTitle = oldEvent.getTitle();
          var oldEventStart = oldEvent.getStartTime();
          var oldEventEnd = oldEvent.getEndTime();
          var oldEventLocation = oldEvent.getLocation();

          if (newEventTitle != oldEventTitle || newEventStart.toISOString() != oldEventStart.toISOString() || newEventEnd.toISOString() != oldEventEnd.toISOString() || newEventLocation != oldEventLocation)  {
          //above wasn't working when attempting to compare dates and strings in one line, so converted the dates to ISO (date formt) sting (https://stackoverflow.com/questions/11174385/compare-two-dates-google-apps-script)

            oldEvent.deleteEvent();

            var newEvent = calendar.createEvent(newEventTitle, newEventStart, newEventEnd,{location:newEventLocation,description:newEventDesc});
            var newEventId = newEvent.getId();
            sheet.getRange(i+2,eventIDCol+1).setValue(newEventId);

            sheet.getRange(i+2,lastActionCol+1).setValue('-Updated-');
            sheet.getRange(i+2,lastActionCol+1).setFontColor('red');
            sheet.getRange(i+2,updateDateCol+1).setValue(dateToday);
             sheet.getRange(i+2,updateDateCol+1).setFontColor('black');

          }

          else {

            sheet.getRange(i+2,lastActionCol+1).setValue('-Unchanged-');
            sheet.getRange(i+2,lastActionCol+1).setFontColor('black');

          }

        }
    }
    numValues++;
  }

//hide updating message
//updateRange.setFontColor('white')

  //Lock written cells
  //var writtenCells_start
  //var writtenCells_end = sheet.getRange(i+2,lastActionCol+1)

}

//add a menu when the spreadsheet is opened
function onOpen() {
  var sheet = SpreadsheetApp.getActiveSpreadsheet();
  var menuEntries = [];
  menuEntries.push({name: "Update Calendar", functionName: "pushToCalendar"});
  sheet.addMenu("Push to Calendar", menuEntries);
}

/*
Notes:
         To trigger an alert use Browser.msgBox();
*/
```
