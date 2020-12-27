# catbot
my goal was to build a solar powered boat that can cruise up and down the Neckar river in Heidelberg.

to plug it all together and make it work below are all the steps i took. i will also list all the components required both in hardware and software with sources where to purchase and download them.

## 3d printing the frame
See files in this repo.

## wiring
1. wire gps and compass to flight controller as described here: http://www.mateksys.com/?portfolio=f405-wing#tab-id-6. for this to work i had to add the typical servo plugs (dupont female) to the end of the provided cable for the gps. I bought a kit for crimping and this made this step really easy. i had not soldered the pins on the flight controller well enough (led-free) which led to initial problems - I will will now use led with led form now on.
1. wire raspberry pi and flight controller gnd-gnd, 5v-5v, tx1 to rx, rx1 to tx with Dupont female to female wires.
1. wire battery and flight controller with a switch on the positive wire to be able to switch everything off with one single switch.
1. solder escs to 2 motor pads on flight controller with a switch allowing to turn power to escs on with delay vs turning power on on the flight controller. this was necessary to work around a problem where when both the flight controller and the escs used were turned at the same type the escs would not initialize correctly as they expected a pwm value around 1500 on initialization.
1. wire battery with solar panel with a waterproof plug.
1. hook up tgy-ia6c receiver via a female to female dupont cable wiring gnd, vcc and ppm to the slots marked Sbs on the f405 wing

## software
1. download and burn rpanion0.7 image to a 16 gb sd card and put sd card into the pi
1. update firmware of thy-i6s transmitter with 2.0.0.58 version from hobbyking website using a usb cable
1. download and install mission planner for windows (yes, qgroundcontrol would also do the job but all docs on the ardupilot side are about mission planner so i bit the bullet and pulled out an old windows laptop my daughter had used in the past)
1. enable udp port 14 withing windows firewall to allow udp traffic coming from the raspberry pi to reach the mission planner app running on Windows
1. download qgroundcontrol on ipad and iphone as an alternative option for gound control when you want to use it mobil
1. hook up f405 via usb cable on computer
1. install drivers if required for f405 board
1. connect to the board (e.g. on COM3)
1. in mission planner install rover 4.0.0 firmware onto the board (it might be that I used a video from painless360 on youtube for an in between step where inav firmware was replaced with ardupilot software)
1. go to data view and check that the horizon is moving when you shake the flight controller

## configuring the firmware parameters part 1 - partially outside
Most of this is done in the ground control software - the instructions here are for mission planner.
1. hook up f405 via usb cable on computer and connect
1. go to mission planner > config > paramter tree view
1. set FRAME_CLASS to 2 (Boat)
1. set SERIAL1_BAUD baud rate to 921 and SERIAL1_PROTOCOL to 2 for Mavlink 2 for telemetry with the pi
1. in mission planner > setup calibrate the accelerometer (doing this inside is ok)
1. go outside on open field for the next steps to calibrate the compass - don't think of trying this inside
1. turn on power from the battery and after5 seconds power to the escs. they should show blue lights and beep both. turning on the power from the battery is necessary to power the gps and the compass.
1. wait for Mateksys m8q-5883 blue led blinking(1Hz) - this means 3D fixed
1. in mission planner wait for GPS 3d lock to show up in mission planner after some time
1. under setup > sensors > calibrate compass - this will require some twisting the device even with usb cable attached
1. change servo output for servo1 to throttleleft and for servo2 to throttleright - this is for skid steering with 2 motors
1. under setup > optional hardware > motor test
1. test motor c at 20% - it should spin to move forward
1. test motor d at 20% - it should spin also forward

## configuring the transmitter part 1
1. turn on power to transmitter
1. factory reset on transmitter
1. on transmitter select rx bind
1. quickly turn off receiver (by unplugging its power line or turning both usb power and battewry power off) and turn it on to bind
1. receiver red led should turn steady
1. in turnigy transmitter set ext battery sensor for 3s battery check
1. in transmitter tie channel 8 to switch a and switch b
1. (in transmitter reverse pitch for channel 2)

## configuring the firmware parameters part 2
1. with mission planner connected via usb go to setup > calibrate remote control
1. in mission planner > setup > main hardware define flight modi for channel 8
  1. hold
  1. manual
  1. loiter
  1. auto
1. check that the flight mode can be changed with switch a and b
1. in mission planner > config > full paramter tree set RC6_OPTION to 41 (disarm/arm with channel 6)
1. check that switch d arms or disarms (might arm only on second attempt if in failsafe mode)
1. in mission planner > config > full parameter tree switch SERVO1_REVERSED to 1
1. in mission planner set BATT_CAPACITY to 8700 mAh
1. in mission planner turn off all failsafe - not needed when having a permanent ground control connection via LTE
1. in mission planner turn off all prearm checks - not needed if the props are covered 
1. set cruise_speed to 1 m/s (checking in operation if its meaningful, its about 3 km/h)
1. in mission planner set RCMAP_PITCH to 3 and rcmap_throttle to 2 to have everything on the right stick on the mode 2

## part 3 - tuning paramater settings
Here the instructions on the ardupilot are pretty important (under first steps)
1. set MOT_THR_MIN to 30 (based on trying in setup > additional hardware > motortest which percentage makes them spin slowly)
1. set ACRO_TURN_RATE to 30 degrees per second (making a turn to find out if it is more or less in your case)
1. set ATC_STR_RAT_FF to 2
1. set PILOT_STEER_TYPE to 3 to not reverse when going backward

## part 4 - misc parameter settings
1. set LOIT_TYPE to 1 to always face bow on loiter to the loiter point
1. set MIS_DONE_BEHAVE to 1 loiter after mission complete to simply loiter after the mission instead of hodl which would let you slip down the river




