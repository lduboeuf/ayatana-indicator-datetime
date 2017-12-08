# Ayatana Indicator DateTime

## ACTIONS

 * "desktop.open-settings-app"
 * "phone.open-settings-app"<br />
   Description: open the settings application.
   State: None
   Parameter: None

 * "desktop.open-alarm-app"
 * "phone.open-alarm-app"<br />
   Description: open the application for creating new alarms.
   State: None
   Parameter: None

 * "desktop.open-calendar-app"
 * "phone.open-calendar-app"<br />
   State: None
   Parameter: int64, a time_t hinting which day/time to show in the planner,
                     or 0 for the current day

 * "desktop.open-appointment"
 * "phone.open-appointment"<br />
   Description: opens an appointment editor to the specified appointment.
   State: None
   Parameter: string, an opaque uid to specify which appointment to use.
                      This uid comes from the menuitems' target values.

 * "set-location"<br />
   Description: Set the current location. This will try to set the current
   timezone to the new location's timezone.<br />
   State: None<br />
   Parameter: a timezone id string followed by a space and location name.
              Example: "America/Chicago Oklahoma City"

 * "calendar"<br />
    Description: set which month/day should be given focus in the indicator's
                 calendar. The planner will look for appointments from this
                 day to the end of the same month.
                 Client code implementing the calendar view should call this
                 when the user clicks on a new day, month, or year.<br />
    State: a dictionary containing these key value/pairs:
          "appointment-days": an array of day-of-month ints. Used by the
                              calendar menuitem to mark appointment days.
          "calendar-day": int64, a time_t. Used by the calendar menuitem
                          to know which year/month should be visible
                          and which day should have the cursor.
          "show-week-numbers": if true, show week numbers in the calendar.<br />
    Parameter: int64, a time_t specifying which year/month should be visible
                      and which day should have the cursor.


## CUSTOM MENUITEMS

 * Calendar
   - x-canonical-type         s "org.ayatana.indicator.calendar"

 * Alarm
   - label                    s short summary of the appointment
   - x-canonical-type         s "org.ayatana.indicator.alarm"
   - x-canonical-time         x the date of the appointment
   - x-canonical-time-format  s strftime format string

 * Appointment
   - label                    s short summary of the appointment
   - x-canonical-type         s "org.ayatana.indicator.appointment"
   - x-canonical-color        s color of the appt's type, to give a visual cue
   - x-canonical-time         x the date of the appointment
   - x-canonical-time-format  s strftime format string

 * Location
   - label                    s the location's name, eg "Oklahoma City"
   - x-canonical-type         s "org.ayatana.indicator.location"
   - x-canonical-timezone     s timezone that the location is in
   - x-canonical-time-format  s strftime format string



## CODE

### Model

  The app's model is represented by the "State" class, and "Menu" objects
  are the corresponding views. "State" is a simple container for various
  properties, and menus connect to those properties' changed() signals to
  know when the view needs to be refreshed.

  As one can see in main.c, the app's very simple flow is to instantiate
  a state and its properties, build menus that correspond to the state,
  and export the menus on DBus.

  Because State is a simple aggregate of its components (such as a "Clock"
  or "Planner" object to get the current time and upcoming appointments,
  respectively), one can plug in live components for production and mock
  components for unit tests. The entire backend can be mix-and-matched by
  adding the desired test-or-production components.

  Start with:<br />
  include/datetime/state.h<br />
  include/datetime/clock.h<br />
  include/datetime/locations.h<br />
  include/datetime/planner.h<br />
  include/datetime/settings.h<br />
  include/datetime/timezones.h<br />

  Implementations:<br />
  include/datetime/settings-live.h<br />
  include/datetime/locations-settings.h<br />
  include/datetime/planner-eds.h<br />
  include/datetime/timezones-live.h<br />

### View

  Menu is a mostly-opaque class to wrap GMenu code. Its subclasses contain
  the per-profile logic of which sections/menuitems to show and which to hide.
  Menus are instantiated via the MenuFactory, which takes a state and profile.

  Actions is a mostly-opaque class to wrap our GActionGroup. Its subclasses
  contain the code that actually executed when an action is triggered (ie,
  LiveActions for production and MockActions for testing).

  Exporter exports the Actions and Menus onto the DBus, and also emits a
  signal if/when the busname is lost so indicator-datetime-service knows
  when to exit.

  include/datetime/menu.h<br />
  include/datetime/actions.h<br />
  include/datetime/exporter.h<br />

