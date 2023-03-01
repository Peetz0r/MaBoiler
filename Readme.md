# MaBoiler

> This is the story of Ma Boiler, the meanest ~~cat~~ water heater in ~~old Chicago town~~ my house.

My boiler is an [Eldom 50L water heater](https://eldominvest.com/en/product/629.html), except that mine came without the Wifi addon board. Actually, mine is a [Green Hat rebranded version](https://www.groenehoedduurzaam.nl/digitale-50-liter-elektrische-boiler.html), but it's really the exact same thing with a different sticker.

I've been using ths boiler completely unconnected, offline, and it has been perfectly fine. But recently I switched to an elecritity provider with dynamic hourly prices. So it makes economical (and environmental) sense to use power when prices are low. Appliances like the washing machine, dryver, and dishwasher all have a 'Delayed Start' feature, which takes care of a lot of my energy usage. But my boiler is the hungriest of my appliances, yet it has no good way to control it externally.

It has a PCB with a microcontroller, a conector for a 10K NTC temperature sensor, a big relay to switch the 2 kW heating element), and an isloated power supply. On the front it has a color LCD and 4 buttons.

## Specs:
* Microcontroller: NXP MCF51QM128
* Relay: Omron G4A-1A-E 12VDC 20A 250VAC (overkill, so I guess it'll last forever. Not cheap either.)
* LCD: ER-TFT018-2 (1.8" 160x128 pixels)
* Temperature sensor: 10K NTC connected via 2-pin JST XH connector, measures 1530 Ω at 75°C
* Heating element: nominal 2 kW resistive element, measures around 26.5 Ω which is almost exactly V²/P = 230²/2000
  * fun fact: the mains voltage in my house is usually between 235 V and 240 V, so the actual power is usually between 2.1 and 2.2 kW.
* STWD100 watchdog IC
* PE301522 230V AC -> 7.5V AC isolated transformer, DB106S bridge rectifier, NCP3170 switching converter, 3.3V
* 0.33F 5.5V supercap (charged up to 3.3V) for the RTC, connected to the main VDD rail
  * I assume the microcontroller somehow detects absence of mains power, and then puts everything in a low power mode but keeps the clock running

## Pinouts:
### Wifi module header
1. GND
2. 3.3 V
3. #44: ADC0_SE22 or more likely UART0_TX
4. #43: ADC0_SE21 or more likely UART0_RX

### [BDM](https://en.wikipedia.org/wiki/Background_debug_mode_interface) header
1. (square) Watchdog Enable (on the STWD100)
2. GND
3. BKGD/MS
4. RESET_b

### Buttons (top to bottom)
* #61: PTF7 **⏻** ⮌
* #60: PTF6 **⮝** t°
* #59: PTF5 **⮟** ☰
* #58: PTF4 **⮨** i

### Other pins:
* #9, PTA2: Relay primary coil


It also has a 4-pin header for a wifi module, but it's not really for sale. I asked the vendor where I got my boiler, but they told me they can't order this part specifically.

Of those 4 pins, 2 of them are power (3.3V and GND), the other 4 pins are probably TTL serial, but no actual communication happens. The current hypothesis is that the wifi module initialises any communication. But I don't have one, so I can't see that in action.

## The options

There are three options:

### Just the module

Either I figure out the protocol in some other way, possibly trough dumping the firmware, and I create a replacement for the wifi module.

If dumping the firmware is possible over the debug port, then this may be the easiest option.

### The entire thing

Or I replace the entire controller with my own design, preferably with the exact same dimensions, and the LCD and buttons in the exact same spots.

This option is definitely feasible, but possibly more work and more expensive than the other. It's also sligtly more risky in terms of reliability and safety.
But it should be fine, since there are a bunch of safely features outside of the PCB: a thermal cutout (96°C+0/-6) on the power inlet, a check valve and pressure relief valve on the water inlet.
But I don't want the water to get anywhere near 96°C. The original firmware has a maximum temperature setting of 75°C, which I'll want to replicate. To make this reliable, I'll have to use the watchdog features of the microcontroller.

### Give up

No. Just no.

Either way I'll end up with a water heater with some kind of connectivity, probably MQTT over wifi. The most likely microcontroller would be an esp32.

I'll try to gather some docs and pictures in `./docs/` this repo, work on the board in `./pcb/`, and probably also work on some firmware at some point in the future.

![Stock PCB annotated, from manual](/docs/stock-pcb-annotated-from-manual.png)

Note: the above picture is taken from the [manual](docs/NHC-56_RM_v2_0_EN.pdf), I do not own the NRM-W3 wifi module.
