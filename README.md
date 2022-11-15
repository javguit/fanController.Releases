# fanController.Releases
This is the releases repository of the fan controller software and board.

# A bit of history:

The project started in late 2019. I had an old X58 system with a x5675 @ 4.3ghz and a NVIDIA 1060 6gb in it. The computer worked fine, but I really didnt like the high temps that the x5675 was generating. I used to use the Almico SpeedFan ( https://www.almico.com/speedfan.php ) program, but its kind of hard to use/understand and it's outadted and no longer recieves updates of the developer. Also I needed more fan headers to control the 2 frontal fans, 2  AIO fans and 1 rear fan.
This coincided with the time I was starting to explore the arduino world. I quickly realized I could build a board to control fans, and thus I created a very basic software to control them.

After that I started playing with OpenHardwareMonitor integration, but when i realized that that project kind of died, switched to LibreHardwareMonitor.
Both these softwares didnt support W36xx boards, so I also had to search for datasheets and add the SuperIO support for the boards that used those chips.


During 2020 lockdown i got the chance to work on adding OHM/LHM support of W36xx, create a very basic board and a software to control it and itegrate them all together. It was really buggy but did the job just fine. After a while when everything started to go back to normal other projects and RL stuff needed more attention and so this project had to wait.


A couple years passed and on October the 26th 2022 I decided to finish the project once and for all and release it to the public.

# Board:

The board is basically an atmega32u4 controlled 4 way buck converter. It uses an arduino pro micro, and some stuff you can easily get in an electronics store and build it yourself. 

There's a catch tho, the COIL WHILE!. The frequency when the 4 channles are controlled (not 0% speed nor 100% speed) per channel is 2225hz ( 2.2khz ). At that frequency the coil while is very audible, so if you want to build this board you'll have to use some glue and neutral silicon sealant (not the one that has vinegar smell, thatone is acidic and will ruin the coils) to muffle it ( or maybe build your own board with a faster ADC!!! ). Also maybe a 3d printed cap will also help ( fill it with sealant and put the coil in it ).

## Create your own board!:
You can also create your own serial controller. You can set which COM port to connect or just let the program pool every port and let it decide whichone to connect.
The 3 basic commands it uses are the following ( all commands are ended with the special string `*EOM*` ):

#### Commands the board will receive from the software:
```
con(port)*EOM*
spd(fan 1 spd)(fan 2 spd) ... (fan n spd)*EOM*
get*EOM*
```

1. `con(port)*EOM*` 

This command is used for the program to get the connection string and lets the program identify this device as a fan controller. This command needs to return a connection string as described below.

2. `spd(fan 1 spd)(fan 2 spd) ... (fan n spd)*EOM*`

  This command sets the fans speeds. The speed value ranges from 0 to 9999, being 0 as 0% speed and 9999 as 100% speed. The program will send this string whenever any fan speed change is detected. So every time a change is detected by the program, it will send all the values to the board despite them having changed or not. This command needs to return an status string as described below.

3. `get*EOM*`

  This command is just to get the status string from the board. This command is sent to the board when no speed change was detected. This is also used as an alive  message to the board, as if it doesnt recieve any message from the program for 10 seconds all the fans will return to 50% speed. This command needs to return an status string as described below.
  
#### Commands the board will send to the software:

1. The connection string the program identifies as valid has to follow this format:

- `390218390218392180.(fan quantity).(port).*EOM*`
  
  - `390218390218392180` is an arbitrary string i chose, so nothing special there.
  - `(fan quantity)` needs to be an integer. Describes how many fans the board can control.
  - `(port)` is the same port that was sent with the `con(port)*EOM*` command. The port has to be a com port, so the valid strings are COM1, COM2, COM3, etc.
  The status string just needs to follow this format:
    
2. The status string the program identifies as valid has to follow this format:

- `(fan 1 debug info),(fan 1 rpm pulses),(fan 1 speed),(fan 2 debug info),(fan 2 rpm),(fan 2 speed), .... ,(fan n debug info),(fan n rpm),(fan n speed)*EOM*`

  - `(fan x debug info)` is used to debug different things of the board. Nothing in this field has an effect on the program. Just keep in mind no to use the comma character.
  - `(fan x rpm pulses)` is the number of hall pulses the board detected. The board reads the pulses for 2 seconds and then stores it to send to the program in the next status message. The program will convert this value to RPM. The conversion the program does is `rpmPulses * 15`
  - `(fan x speed)` is used to debug the boards fan speed. Nothing in this field has an effect on the program. Just keep in mind no to use the comma character.

3. `error*EOM*`

    If the board can't identify the message then it should return this error message.

#### Example on COM communication flow with a 4 fan controller board (viewed from the boards perspective):
1. Software to Board: `conCOM14*EOM*`

2. Board to Software: `390218390218392180.4.COM14.*EOM*`

3. Software to Board: `spd9999500070008500*EOM*`   // speeds: Fan 1 - 9999 , Fan 2 - 5000, Fan 3 - 7000, Fan 4 - 8500

4. Board to Software: `debuginfo,212,9999,debuginfo,68,5000,debuginfo,89,7000,debuginfo,164,8500*EOM*` // RPM: Fan 1 - 3180 , Fan 2 - 1020, Fan 3 - 1335, Fan 4 - 2460

5. Software to Board: `get*EOM*`

6. Board to Software: `debuginfo,212,9999,debuginfo,68,5000,debuginfo,89,7000,debuginfo,164,8500*EOM*` // RPM: Fan 1 - 3180 , Fan 2 - 1020, Fan 3 - 1335, Fan 4 - 2460

7. And so on towards infinity.
