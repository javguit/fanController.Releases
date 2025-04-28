# fanController.Releases
This is the releases repository of the fan controller software.
Here you'll find the link for the board schematic, pcb and sourcecode, and the STL files of the pcibracket+inductors cap.<br>
<b>The board is optional! You can use the software without the board!</b>

[![Github All Releases](https://img.shields.io/badge/Download-fanController-green)](https://github.com/javguit/fanController.Releases/releases/latest) [![Github All Releases](https://img.shields.io/github/downloads/javguit/fanController.Releases/total.svg)]() [![Donations Badge](https://img.shields.io/badge/Donate-%3AD-blue)](https://github.com/javguit/fanController.Releases/blob/main/README.md#donate) [![License Badge](https://img.shields.io/badge/License-CC0-orange)](https://github.com/javguit/fanController.Releases/blob/main/LICENSE)

# Software:

The software relays on LHM's library to get the computer's temperatures and RPMs and set the fans speeds. In addition to it, you can set custom curves, attach different temp sensors to a fan, set automatic/manual/computer control, set min/max speeds, off temps, start speeds, min speed for RPM read (many fans return messed up rpm reads when the speed is too low), test your configuration by manually changing the temps, save and reutilize your curves, export curves to share them.

To better understand how to use the software please visit this site's Wiki (still in construction).

<b>IF THERE'S A TEMP SENSOR, A FAN CONTROL OR AN RPM SENSOR THAT DOESN'T WORK PLEASE REFER THAT TO LHM'S GITHUB</b>.

![Screenshot](/images/main-COMdevice.jpg) <br>
![Screenshot](/images/auto-config3.jpg)

## Prerequisites:

- .NET Framework 4.7.2 

## Installation:

It does not need an installation. Just unzip the folder, place it where you want and run the executable.

## Uninstallation:

Just close the program and delete the folder. Don't forget to remove the autorun option from the settings menu, so the task in the task scheduler gets erased too.

## Libraries used:

The libraries used are:

1. Libre Hardware Monitor: https://github.com/LibreHardwareMonitor/LibreHardwareMonitor
2. Newtonsoft JSON: https://github.com/JamesNK/Newtonsoft.Json
3. TaskScheduler: https://github.com/dahall/taskscheduler
4. HidSharp ( dependency of LHM ): https://www.nuget.org/packages/HidSharp

# Board:

<b>The board is optional! You can use the software without the board!</b>

Schematic, pcb and source code: https://oshwlab.com/javguit/fan-controller-2-5-3pin-4#P2 <br>
STL files for 3dprinting the pci bracket: https://www.thingiverse.com/thing:5639938

The board is basically an Arduino Pro Micro ( atmega32u4 ) controlled 4 channel step down voltage regulator. It uses an arduino pro micro, and some stuff you can easily get in an electronics store and build it yourself. This board is prepared to control 4 3-pin fans (also 4-pin fans since this is controlling the voltage). Adapting this to a 4-pin only board ( or combine 2 and 2 for example ) should be pretty straight forward since 4-pin fans are easier to control and hence easier to build.

Why use an Arduino Pro Micro and not an uber speciallized controller? Because this is a DIY project, and a Pro Micro is easier to source and program.

Now, the coil noise. This board is not particularly fast when we examine the ADC and the PWM speeds. Even tho you can change the prescalers and tinker a little bit with the PWM registers, the fact is that this board is not the best choice when building a stepdown converter ( and its even less suitable for 4 step down converters ). This "slowness" generates noise when the current hits the coils, thus generatingthe famous coil whine ( its not really a whine in this case, but a really low noise ). So if you want to build this board you'll have to apply superglue ( it has to be the liquid one, the gel one is too vicous ) on the coil's copper wire and when its dry apply neutral silicon sealant (not the one that has vinegar smell, that one is acidic and will ruin the coils ) to muffle it. Also a 3d printed cap may help ( fill it with sealant and put it over the coils ).

![Screenshot](/images/board.uncap.jpg) <br>
![Screenshot](/images/board.cap.jpg)

## Create your own board!:

You can also create your own serial controller, or maybe with a better ADC and PWM. You can set which COM port to connect or just let the program pool every port and let it decide which one to connect.  The serial speed has to be 115200 bits/s. <br>
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
  
  - `390218390218392180` is an arbitrary string I chose so the program recognizes this COM device as a valid controller.
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

# Donate!:

If you want to support this project, you can do so by sending cryptocurrency to the following wallets.

BTC: 1BMcMP6yEHwJkpKRGuLEfFh2o7JpLFpaz5 <br>
ETH (erc20): 0xdd9d08dbaa9324aabac570c5d0e67e79f5b92fbe <br>
USDT (erc20): 0xdd9d08dbaa9324aabac570c5d0e67e79f5b92fbe <br>
SOL (SOL): ACJVh741zxjMrLtw2jQeWGhnraeCsSsnhAV2mM48Daqa

THANK YOU VERY MUCH!

# A bit of history:

The project started in late 2019. I had an old X58 system with a x5675 @ 4.3ghz and a NVIDIA 1060 6gb in it. The computer worked fine, but I really didnt like the high temps that the x5675 was generating. I used to use the Almico SpeedFan ( https://www.almico.com/speedfan.php ) program, but its kind of hard to use/understand and it's outadted and no longer recieves updates of the developer. Also I needed more fan headers to control the 2 frontal fans, 2  AIO fans and 1 rear fan.
This coincided with the time I was starting to explore the arduino world. 

After that I started playing with OpenHardwareMonitor integration, but when I realized that that project kind of died, switched to LibreHardwareMonitor.
Both these softwares didnt support W36xx boards, so I also had to search for datasheets and add the SuperIO support for the boards that used those chips.

During 2020 lockdown i got the chance to work on adding OHM/LHM support of W36xx, create a very basic board and a software to control it and itegrate them all together. It was really buggy but did the job just fine. After a while when everything started to go back to normal ( COVIDwise ) other projects and RL stuff needed more attention and so this project had to wait.

A couple years passed and on October the 26th 2022 I decided to finish the project once and for all and release it to the public.
