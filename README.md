# catbot
my goal was to build a solar powered boat that can cruise up and down the Neckar river in Heidelberg.

to plug it all together and make it work below are all the steps i took. i will also list all the components required both in hardware and software with sources where to purchase and download them.

videos and other projects are described here: http://4xb.de

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
1. set WP_PIVOT_ANGLE to 100 to avoid that with the default of 60 and a lot of wind or current the pivot turn kicks in too early

## part 4 - misc parameter settings
1. set LOIT_TYPE to 1 to always face bow on loiter to the loiter point
1. set MIS_DONE_BEHAVE to 1 loiter after mission complete to simply loiter after the mission instead of hodl which would let you slip down the river

## configuring wifi connectivity
1. download and burn rpanion0.7 image to a 16 gb sd card and put sd card into the pi
1. start up the power on the pi and the rest of the devices
1. connect to the rpanion wifi opened by the pi with passcode rpanion123 from any computer
1. open up the rpanion server web ui at 10.0.2.100:3000 in a browser
1. bring up the flight controller tab
1. change the baud rate from 9600 to 921600 and the protocol to mavlink 2.0
1. start telemetry > you should see packets streaming in from the flight controller
1. connect the computer with your gcs software to the rpanion wifi network
1. find out the ip address of your gcs computer in the wifi settings
1. go back to the rpanion server ui in the browser and configure the ip address and port under the tab flight controller. if your ip address is for example 10.0.2.171 then put in a new output at 10.0.2.171:14550. this will work for both mission planner and qgroundcontrol as the default port. repeat this for all other devices you want to use gcs software on.
1. turn on video streaming in the rpanion server ui and take note of the rtsp streaming address. for me it only worked when entering it in qgroundcontrol. setup with gstreamer with mission planner was not possible to get to work.

## configuring 3g connectivity
1. get yourself an account and create a new (virtual) network at my.zerotier.com. this is necessary as otherwise you cannot connect the ground control software and the companion computer aka pi anymore as they will be not in the same (local) network anymore and zerotier will create a virtual private network where they seem to be in the same network to all applications but no one else cget into the vpn if you do not let them.
1. download the windows, mac, android or ios zerotier app from the zerotier site to the device on which you want to run the ground control station. to not confuse things i chose another device then the one i'm connecting with via wifi.
1. join your network from this device by entering the network id and in zerotier central authorize the device. it should show up in zerotier central as "online" 
1. download linux version of zero tier from command line with this command: "curl -s https://install.zerotier.com | sudo bash"
zerotier-cli join with your zero tier network id
1. configure usb stick to not use a pin anymore on a mac or windows pc
1. plug usb stick into pi
1. it should turn cyan, blue or green steady
1. pi should now have internet via 3g
1. connect via ssh to the pi from e.g. a Mac or using putty on the PC
1. login with user pi and password raspberry
1. check that internet is there by doing a wget or ping on any web address (e.g. ping 4xb.de)
1. download and install zerotier (it will be downloaded, installed and automatically started and will also automatically start on next boot) using this command: "curl -s https://install.zerotier.com | sudo bash" from the command line of the pi
1. the next thing to do to bring the pi into the vpn is to join your zerotier network id by entering "sudo zerotier-cli join 9872387..." putting your network id there instead of 9872387...
1. now you need to also authorize this device in the zerotier central console at my.zerotier.com. once authorized it will get its own manual ip address in your network. now the pi can be reached from the gcs and vice versa using the ip addresses listed in the zerotier central page.
1. on the computer used for the gcs you can now bring up the rpanion server ui by using the manual ip from your zerotier vpn network appended by ":3000".
1. in this ui you can now add the zerotier vpn ip address of the companion computer aka raspberry pi appended with ":14550" to the list of routes in the flight controller section - this will make the mavlink flight data to be also now forwarded via the vpn
1. now it should be possible in mission planner on windows or in qgroundcontrol on mac or ios wherever you want to use your gcs to connect to the pi via 3g. i used an iphone which i can put on top of my remote control.
1. you can also configure the video stream but make sure you know the amount of data pushed is ok with your mobile plan and also be aware of the video data competing with the mavlink data on the same connection.


