# Digital Guitar Pedal

This is an ongoing project for a digital signal processing system for electric guitars.
It provides multiple effects which can be individually adjusted using four knobs.

## Motivation

I had seen several job postings from certain large engineering firms that included "ARM Cortex" among the qualifications and preferred skills;
I decided that having that technology on my resume would be great for my career prospects.
Moreover, signal processing, particularly digital signal processing, is really interesting to me,
so I wanted to build a digital signal processing system that used an ARM processor.

## Goals

##### Low Noise and Distortion
Electric guitar pedal enthusiasts tend to hold digital audio processing technology in suspicion, preferring the supposed purity of analog technology.
Therefore, my first goal was to minimize noise and distortion.

##### Low Power Consumption
Guitarists also like to chain several pedals together, but the power supply can only supply enough power for so many pedals at once;
the more power a particular pedal draws, the fewer other pedals the guitarist can add to the chain.
Digital pedals tend to consume more power than analog pedals, so minimizing the power draw of this pedal is a much larger concern of mine compared to the analog pedals I have designed.

## Hardware Overview

### Parts Selection

#### STM32H7A3

The STM32H7A3 is from STMicroelectronics H7-series of high-performance microcontrollers.
I was originally considering the older F4-series and G4-series, 
but I decided that if I am going through the effort of developing the hardware for a DSP system, 
I should just get a much nicer and more modern controller, so that I would never be bottlenecked by the hardware.

The STM32H7A3 is an ARM Cortex-M7 processor, which, as stated above, is a technology I wanted to put on my resume.
In addition to this, STM32 microcontrollers in general are very popular among hobbyists, so there should be a lot of documentation and tutorials available online.

#### PCM3060

The PCM3060 is a hardware codec (meaning it incorporates an ADC and a DAC in one chip) from Texas Instruments.
The built-in ADC and DAC on the STM32H7A3 are 12-bits and 10-bits, respectively, meaning they did not have high enough resolution to be used for audio signal processing.
Like the STM32H7A3, the PCM3060 is popular among hobbyists, so there should be plenty of documentation online.

### Power Scheme

The power scheme of the hardware is of the utmost importance to both of my stated goals with this project.
It wouldn't matter if I could procure ideal amplifiers with no noise whatsoever, if the power rails themselves were noisy.
I decided to combine both switching regulators and LDO regulators to reap the benefits of both.
The only downsides of this approach would be the increased component count and additional board complexity/space;
but since I put a 144-pin chip on this board, a couple extra small chips didn't bother me.

The power scheme includes two main power rails: the digital 3.3V rail and the analog 5V rail.
The digital rail supplies both the STM32H7A3 and the PCM3060 digital VDD;
the analog rail supplies the PCM3060 analog VCC and the input/output op-amp buffers.
The digital rail is created with only a single switching regulator (AP5100) bucking the 9V input directly down to 3.3V.
The analog rail is created with a switching regulator (AP5100) bucking the 9V down to approximately 5.9V, and a LDO regulator (MCP1754) shifting it down to a very clean 5V.

In addition to the power rails, I configured the STM32H7A3 to bypass the internal LDO, only using the internal SMPS to supply its core voltage, thus reducing the microcontroller's power consumption considerably.
I also cut out the analog supply to the STM32H7A3.
I was able to do both these things because I offloaded all analog functions to the PCM3060, thus eliminating the need for a really clean power supply to the STM32H7A3.

### User Interface

The board includes a footswitch, four rotary encoders, and a small character-only display.
The footswitch enables/disables then entire effect; if pressed down, the audio signal bypasses the entire codec and microcontroller, instead being routed through a single op-amp buffer.
The four rotary encoders act as generic knobs which control different parameters for each effect.
The display shows the current effect and current parameter being adjusted.

## Software Overview

Coming soon.