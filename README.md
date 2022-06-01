# Throughhole_Braided_DAC
Improvements to the DAC code for the Braids Through-hole – SoundForce module

This repository was born after building the Braids Through-hole – SoundForce module which is a derrivative work of Mutable Instruments Braids by Emilie Gillet with original firmware: https://github.com/pichenettes/eurorack/tree/master/braids and also the with changes for the derivative firmware by Tim Churches: https://github.com/timchurches/Mutated-Mutables

On completion of the Braids through hole it was noticed that the DAC output was very noisy. It sounded like it was dropping samples. The selection of the oscillator caused the noise to change character.

Investigation revealed a few issues:
1) The MCP4822 chipselect CS was been driven slow to load each new sample in a few clock cycles of the STM32 and this period was too short. The CS line has to be held low for 40ns before loading a new sample (tCHS) as shown in figure 1-1: https://www.mouser.co.uk/datasheet/2/268/21953a-8929.pdf
Adding extra NOP instructions allows the timinig to be made more precise and allows a longer CS low time to be defined.
2) The SPI lines were set to 50MHz, however at this speed the lines interfere with each other especially crossing the inline connections from the STM32 bluepill to the main board. Dropping to 10MHz greatly reduced the number of dropped samples. 
3) The lines to the display were also set to 50MHz, however the display can operate with 2MHz. Selecting a slower output line adds extra capacitance and reduces the likelihood that the IO line will induce external interference across the board.


Additionally the author wanted to extend the DAC bit rate beyond 12bits. Fortunately the MCP4822 has two 12bit DAC channels. Using two DACs in parallel with differing size output resistors into a potential divider allows the chip to function up to 24bits. However the current firmware uses 16bit buffers which is adequate for the application. It was decided to aim for 20 bits possible accuracy. This is achieved in hardware using a 10 Mohm resistor connected from pin 6 (VOUTB) of the MCP4822 to C21. VoutA is connected to the same capacitor via 39k. The ratio of 10Mohms/39Kohms is 256.4, so allows DACB to contribute approximately 1/256th of the voltage to the signal generated by DACA. The 16 bits can therefore be divided in software between the two DAC channels to emulate a single 16 bit DAC. The same combination of two resistors could be used to output 20 bits. This could be useful for example for interpolation between samples.

Additionally the DAC has linearity issues as defined in the datasheet for values between 0V and 0.01V. To improve linearity, channel A is drived with 11 bits across the 1.024V to 3.072V range and channel B is driven with 5 bits across a smaller range centred about 2.048V.

This repository provides the modified files for fixing the DAC timings, changing the SPI and display timing and also for generating the dual DAC output for use as 16 bit output. 
