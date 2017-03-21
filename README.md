![SwiftyGPIO](https://github.com/uraimo/SwiftyGPIO/raw/master/logo.png)

**A Swift library to interact with Linux GPIOs/SPI/PWM, blinking leds and much more!**

<p>
<img src="https://img.shields.io/badge/os-linux-green.svg?style=flat" alt="Linux-only" />
<a href="https://developer.apple.com/swift"><img src="https://img.shields.io/badge/swift3-compatible-orange.svg?style=flat" alt="Swift 3 compatible" /></a>
<a href="https://raw.githubusercontent.com/uraimo/SwiftyGPIO/master/LICENSE"><img src="http://img.shields.io/badge/license-MIT-blue.svg?style=flat" alt="License: MIT" /></a>

![](banner.jpg)

## Summary

This library provides an easy way to interact with external sensors and devices using digital GPIOs, SPI interfaces and PWM signals with Swift on Linux.

You'll be able to configure port attributes (direction,edge,active low), read/write the current GPIOs value, use the SPI interfaces (via hardware if your board provides them or using software big-banging SPI) and generate a PWM to drive external displays, servos, leds and more complex sensors.

The library is built to run **exclusively on Linux ARM Boards** (RaspberryPis, BeagleBone Black, UDOO, Tegra, CHIP, etc...) with accessible GPIOs.

**Since version 0.8 SwiftyGPIO targets Swift 3.0, for Swift 2.x [refer to the specific branch](https://github.com/uraimo/SwiftyGPIO/tree/swift-2.2) for sources and documentation.**

##### Content:
- [Supported Boards](#supported-boards)
- [Installation](#installation)
- [Your First Project: Blinking Leds And Sensors](#your-first-project-blinking-leds-and-sensors)
- [Usage](#usage)
    - [GPIO](#gpio)
    - [SPI](#spi)
    - [PWM](#pwm)
- [Examples](#examples)
- [Built with SwiftyGPIO](#built-with-swiftygpio)
    - [Device Libraries](#libraries)
    - [Awesome Projects](#awesome-projects)
- [Additional documentation](#additional-documentation)


## Supported Boards

Tested:
* C.H.I.P.
* BeagleBone Black (Thanks to [@hpux735](https://twitter.com/hpux735))
* Raspberry Pi 2 (Thanks to [@iachievedit](https://twitter.com/iachievedit))
* Raspberry Pi 3
* Raspberry Pi Zero (Thanks to [@MacmeDan](https://twitter.com/MacmeDan))
* Raspberry Pi A,B Revision 1
* Raspberry Pi A,B Revision 2
* Raspberry Pi A+, B+
* OrangePi (Thanks to [@colemancda](https://github.com/colemancda))
* OrangePi Zero (Thanks to [@eugeniobaglieri](https://github.com/eugeniobaglieri)) 
* UDOOs

Not tested but they should work(basically everything that has an ARMv7/Ubuntu14/Raspbian or an ARMv6/Raspbian):
* BananaPi
* OLinuXinos
* ODROIDs
* Cubieboards
* Tegra Jetson TK1


## Installation

To use this library, you'll need a Linux ARM(ARMv7 or ARMv6) board with Swift 3+.

If you have a RaspberryPi (A,B,A+,B+,Zero,ZeroW,2,3) with Ubuntu or Raspbian, get Swift 3.0.2 from [here](https://www.uraimo.com/2016/12/30/Swift-3-0-2-for-raspberrypi-zero-1-2-3/) or follow the instruction from the post and the linked build scripts repository (or these: [1](http://mistercameron.com/2016/06/compile-swift-3-0-on-your-arm-computer/), [2](https://medium.com/@MissionKao/how-to-compile-swift-on-raspberry-pi-ae33e417a61e#.dweiw55iu), [3](http://saygoodnight.com/2016/05/08/building-swift-for-armv6.html), [4](http://morimori.tokyo/2016/02/09/compiling-swift-on-a-raspberry-pi-2-february-2016-update-and-a-script-to-clone-and-build-open-source-swift/)) to build it yourself.

Recent precompiled ARMv7 binaries are also available from [Joe build server](http://dev.iachieved.it/iachievedit/swift-for-arm-systems/) and you can find them [here](http://swift-arm.ddns.net/job/Swift-3.0-Pi3-ARM-Incremental/lastSuccessfulBuild/artifact/), all built for Ubuntu 16.04 from the master repo.

The same Ubuntu binaries could work for BeagleBoneBlack, C.H.I.P. or any other ARMv6/ARMv7 board too when used with the same release.

If your version of Swift supports the SPM, you just need to add SwiftyGPIO as a dependency in your `Package.swift`:

```swift
let package = Package(
    name: "MyProject",
    dependencies: [
        .Package(url: "https://github.com/uraimo/SwiftyGPIO.git", majorVersion: 0),
        ...
    ]
    ...
)
```
And then build with `swift build`.

The compiler will create an executable under `.build/`.

If your version of Swift does not support the Swift Package Manager, download manually all the needed files: 

    wget https://raw.githubusercontent.com/uraimo/SwiftyGPIO/master/Sources/SwiftyGPIO.swift https://raw.githubusercontent.com/uraimo/SwiftyGPIO/master/Sources/Presets.swift https://raw.githubusercontent.com/uraimo/SwiftyGPIO/master/Sources/SunXi.swift https://raw.githubusercontent.com/uraimo/SwiftyGPIO/master/Sources/SPI.swift https://raw.githubusercontent.com/uraimo/SwiftyGPIO/master/Sources/PWM.swift

And once downloaded, in the same directory create an additional file that will contain the code of your application named `main.swift`. 

When your code is ready, compile it (the PWM and SPI files can be deleted if you don't need them) with:

    swiftc *.swift
    
The compiler will create a **main** executable.

**IMPORTANT:** As everything interacting with GPIOs via sysfs/mmapped registers, if your OS does not come with a predefined user group to access these functionalities, you'll need to run your application with root privileges using `sudo ./main`. If you are using a RaspberryPi with a recent Raspbian (post November 2016) or a recent Ubuntu (from 16.04 Xenial onward), this will be not required, just launch your application with `./main`. On misconfigured systems, features like the listeners may require root privileges anyway.

Alternatively, a specific user group for gpio access can be configured manually as shown [here](https://arcanesciencelab.wordpress.com/2016/03/31/running-rpi3-applications-that-use-gpio-without-being-root/) or in this [answer on stackoverflow](https://stackoverflow.com/questions/30938991/access-gpio-sys-class-gpio-as-non-root/30940526#30940526).
After following those instruction, remember to add your user (e.g. pi) to the gpio group with `sudo usermod -aG gpio pi` and to reboot so that the changes you made are applied.

<a href="#first"></a>
## Your First Project: Blinking leds and sensors

If you prefer starting with a real project instead of just reading documentation, more than a few tutorials are available online.

If you are using Swift 3.0 and the latest version of SwiftyGPIO, [Cameron Perry has a great step by step guide](http://mistercameron.com/2016/06/accessing-raspberry-pi-gpio-pins-with-swift/) on how to setup a Raspberry Pi for Swift and using a led and a temperature sensor. 

If you are still using Swift 2.x and need a practical example of how to use SwiftyGPIO (get it from [the specific 2.x branch](https://github.com/uraimo/SwiftyGPIO/tree/swift-2.2)), Joe from iachievedit has written a [fantastic tutorial](http://dev.iachieved.it/iachievedit/raspberry-pi-2-gpio-with-swiftygpio/) that will explain everything you need to know.

Additional tutorials are also available in [中文](http://swift.gg/2016/04/01/raspberry-pi-2-gpio-with-swiftygpio/), [日本語](https://ja.ngs.io/2016/06/01/swifty-gpio/) and [Tiếng Việt](https://techmaster.vn/posts/34237/lap-trinh-swift-tren-raspberry-pi).

## Usage

Currently, SwiftyGPIO expose GPIOs, SPIs(if not available a bit-banging VirtualSPI can be created) and PWMs, let's see how to use them.

### GPIO

Let's suppose we are using a Raspberry 2 board and have a led connected between the GPIO pin P2 (possibly with a resistance of 1K Ohm or so) and GND and we want to turn it on.

First, we need to retrieve the list of GPIOs available on the board and get a reference to the one we want to modify:

```swift
let gpios = SwiftyGPIO.GPIOs(for:.RaspberryPi2)
var gp = gpios[.P2]!
```

The following are the possible values for the predefined boards:
    
* .RaspberryPiRev1 (Pi A,B Revision 1, pre-2012, 26 pin header)
* .RaspberryPiRev2 (Pi A,B Revision 2, post-2012, 26 pin header) 
* .RaspberryPiPlusZero (Raspberry Pi A+ and B+, Raspberry Zero, all with a 40 pin header)
* .RaspberryPi2 (Raspberry Pi 2 or 3 with a 40 pin header)
* .BeagleBoneBlack (BeagleBone Black)
* .CHIP (the $9 C.H.I.P. computer).
* .BananaPi (RaspberryPi clone)
* .OrangePi

The map returned by `GPIOs(for:)` contains all the GPIOs of a specific board as described by [these diagrams](https://github.com/uraimo/SwiftyGPIO/wiki/GPIO-Pinout). 

Alternatively, if our board is not supported, each single GPIO object can be instantiated manually, using its SysFS GPIO Id:

```swift
var gp = GPIO(name: "P2",id: 2)  // User defined name and GPIO Id
```
    
The next step is configuring the port direction, that can be either `GPIODirection.IN` or `GPIODirection.OUT`, in this case we'll choose .OUT:

```swift
gp.direction = .OUT
```

Then we'll change the pin value to the HIGH value "1":

```swift
gp.value = 1
```

That's it, the led will turn on.

Now, suppose we have a switch connected to P2 instead, to read the value coming in the P2 port, the direction must be configured as `.IN` and the value can be read from the `value` property:

```swift
gp.direction = .IN
let current = gp.value
```

The other properties available on the GPIO object (edge,active low) refer to the additional attributes of the GPIO that can be configured but you will not need them most of the times. For a detailed description refer to the [kernel documentation](https://www.kernel.org/doc/Documentation/gpio/sysfs.txt)

GPIOs also support the execution of closures when the value of the pin changes. Closures can be added with `onRaising` (the pin value changed from 0 to 1), `onFalling` (the value changed from 1 to 0) and `onChange` (the value simply changed from the previous one):

```swift
let gpios = SwiftyGPIO.GPIOs(for:.RaspberryPi2)
var gp = gpios[.P2]!


gp.onRaising{
    gpio in
    print("Transition to 1, current value:" + String(gpio.value))
}
gp.onFalling{
    gpio in
    print("Transition to 0, current value:" + String(gpio.value))
}
gp.onChange{
    gpio in
    gpio.clearListeners()
    print("The value changed, current value:" + String(gpio.value))
}  
```

The closure receives as its only parameter a reference to the GPIO object that has been updated so that you don't need to use the external variable.
Calling `clearListeners()` removes all the closures listening for changes and disables the changes handler.
While GPIOs are checked for updates, the `direction` of the pin cannot be changed (and configured as `.IN`), but once the listeners have been cleared, either inside the closure or somewhere else, you are free to modify it.
 

### SPI

If your board has SPI connections and SwiftyGPIO has them among its presets, a list of the available SPIs can be retrieved invoking `hardwareSPIs(for:)` (or `getHardwareSPIsForBoard` for Swift 2.x) with one of the predefined boards.

On RaspberryPi and other boards the hardware SPI SysFS interface is not enabled by default, check out the setup guide on [wiki](https://github.com/uraimo/SwiftyGPIO/wiki/Enabling-SPI-on-RaspberryPi-and-others).

Let's see some examples using a RaspberryPi 2 that has one bidirectional SPI, managed by SwiftyGPIO as two mono-directional SPIObjects:
 
```swift
let spis = SwiftyGPIO.hardwareSPIs(for:.RaspberryPi2)
var spi = spis?[0]
```

The first item returned is the output channel and this can be verified invoking the method `isOut` on the `SPIObject`.

Alternatively, we can create a software SPI using two GPIOs, one that will serve as clock pin and the other will be used to send the actual data. This kind of bit-banging SPI is slower than the hardware one, so, the recommended approach is to use hardware SPIs when available.

To create a software SPI, just retrieve two pins and create a `VirtualSPI` object:
```swift
let gpios = SwiftyGPIO.GPIOs(for:.RaspberryPi2)
var sclk = gpios[.P2]!
var dnmosi = gpios[.P3]!
var spi = VirtualSPI(dataGPIO:dnmosi,clockGPIO:sclk) 
```

Both objects implement the same `SPIObject` protocol and so provide the same methods.
To distinguish between hardware and software SPIObjects, use the `isHardware` method.

To send one or more byte over a SPI, use the `sendData` method.
In its simplest form it just needs an array of UInt8 as parameter:

```swift
spi?.sendData([UInt(42)])
```

But for software SPIs (for now, these values are ignored when using a hardware SPI) you can also specify the preferred byte ordering (MSB,LSB) and the delay between two successive bits (clock width, default 0):

```swift
spi?.sendData([UInt(42)], order:.LSBFIRST, clockDelayUsec:1000)
```

### PWM

PWM output signals can be used for example to drive servo motors, RGB leds and other devices, or more in general, to approximate analog output values when you only have digital GPIO ports.

If your board has PWM ports and is supported (at the moment only RaspberryPi boards), retrieve the available `PWMOutput` objects with the `hardwarePWMs` factory method:

```swift
let pwms = SwiftyGPIO.hardwarePWMs(for:.RaspberryPi2)!
let pwm = (pwms[0]?[.P18])!
```

This method returns all the ports that support the PWM function, grouped by the PWM channel that controls them. 

You'll be able to use only one port per channel and considering that the Raspberries have two channels, you'll be able to use two PWM outputs at the same time, for example GPIO12 and GPIO13 or GPIO18 and GPIO19.

Once you've retrieved the `PWMOutput` for the port you plan to use you need to initialize it to select the PWM function. On this kind of boards, each port can have more than one function (simple GPIO, SPI, PWM, etc...) and you can choose the function you want configuring dedicated registers.

```swift
pwm.initPWM()
```

To start the PWM signal call `startPWM` providing the period in nanoseconds (if you have the frequency convert it with 1/frequency) and the duty cycle as a percentage:

```swift
print("PWM from GPIO18 with 500ns period and 50% duty cycle")
pwm.startPWM(period: 500, duty: 50)
```

Once you call this method, the PWM subsystem of the ARM SoC will start generating the signal, you don't need to do anything else and your program will continue to execute, you could insert a `sleep(seconds)` here if you just want to wait.

And when you want to stop the PWM signal call the `stopPWM()` method:

```swift
pwm.stopPWM()
```

If you want to change the signal being generated, you don't need to stop the previous one, just call `startPWM` with different parameters.

This feature uses the M/S algorithm and has been tested with signals with a period in a range from 300ns to 200ms, generating a signal outside of this range could lead to excessive jitter that could not be acceptable for some applications. If you need to generate a signal near to the extremes of that range and have an oscilloscope at hand, always verify if the resulting signal is good enough for what you need.

## Examples

Examples for different boards are available in the *Examples* directory, you can just start from there modifying one of those.

The following example, built to run on the C.H.I.P. board, shows the current value of all the GPIO0 attributes, changes direction and value and then shows again a recap of the attributes:

```Swift
let gpios = SwiftyGPIO.GPIOs(for:.CHIP)
var gp0 = gpios[.P0]!
print("Current Status")
print("Direction: "+gp0.direction.rawValue)
print("Edge: "+gp0.edge.rawValue)
print("Active Low: "+String(gp0.activeLow))
print("Value: "+String(gp0.value))

gp0.direction = .OUT
gp0.value = 1

print("New Status")
print("Direction: "+gp0.direction.rawValue)
print("Edge: "+gp0.edge.rawValue)
print("Active Low: "+String(gp0.activeLow))
print("Value: "+String(gp0.value))
```

This second example makes a led blink with a frequency of 150ms:

```Swift
import Glibc

let gpios = SwiftyGPIO.GPIOs(for:.CHIP)
var gp0 = gpios[.P0]!
gp0.direction = .OUT

repeat{
	gp0.value = (gp0.value == 0) ? 1 : 0
	usleep(150*1000)
}while(true) 
```

We can't test the hardware SPI with the CHIP but SwiftyGPIO also provide a bit banging software implementation of a SPI interface, you just need two GPIOs to initialize it:

```Swift
let gpios = SwiftyGPIO.GPIOs(for:.CHIP)
var sclk = gpios[.P0]!
var dnmosi = gpios[.P1]!

var spi = VirtualSPI(dataGPIO:dnmosi,clockGPIO:sclk) 

pi.sendData([UInt8(truncatingBitPattern:0x9F)]) 
```

Notice that we are converting the 0x9F `Int` using the constructor `UInt8(truncatingBitPattern:)`, that in this case it's not actually needed, but it's recommended for every user-provided or calculated integer because Swift does not support implicit truncation for conversion to smaller integer types, it will just crash if the `Int` you are trying to convert does not fit in a `UInt8`.

## Built with SwiftyGPIO

A few projects and libraries built using SwiftyGPIO. Have you built something that you want to share? Let me know!

### Libraries
* [Nokia5110(PCD8544) LCD Library](http://github.com/uraimo/5110lcd_pcd8544.swift) - Show text and graphics on a Nokia 3110/5110 LCD display.
* [HD44780U Character LCD Library](https://github.com/uraimo/HD44780CharacterLCD.swift) - Show text on character LCDs controlled by the HD44780 or one of its clones.
* [DHTxx Temperature Sensor Library](https://github.com/pj4533/dhtxx) - Read temperature and humidity values from sensors of the DHT family (DHT11, DHT22, AM2303).
* [SG90 Servo Motor Library](https://github.com/uraimo/SG90Servo.swift) - Drives a SG90 servo motor via PWM but can be easily modified to use other kind of servos.

### Awesome Projects 
* [Portable Wifi Monitor in Swift](http://saygoodnight.com/2016/04/05/portable-wifimon-raspberrypi.html) - A battery powered wifi signal monitor to map your wifi coverage.
* [Temperature & Humidity Monitor in Swift](http://saygoodnight.com/2016/04/13/swift-temperature-raspberrypi.html) - A temperature monitor with a Raspberry Pi and an AM2302.
* [Motion Detector with Swift and a Beaglebone Black](http://myroboticadventure.blogspot.it/2016/04/beaglebone-black-motion-detector-with.html) - A motion detector built with a BBB using a HC-SR502 sensor.
* [DS18B20 Temperature Sensor with Swift](http://mistercameron.com/2016/06/accessing-raspberry-pi-gpio-pins-with-swift/) - Step by step project to read temperature values from a DS18B20 sensor.
* [Swifty Buzz](https://github.com/DigitalTools/SwiftyBuzz) - Swifty tunes with a buzzer connected to a GPIO.
 

## Additional documentation

Additional documentation can be found in the `docs` directory.
