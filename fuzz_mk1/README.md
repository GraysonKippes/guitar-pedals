# Fuzzy Mk. 1
## Introduction
A fuzz pedal is a guitar pedal that distorts a signal by amplifying it and limiting it, cutting off the peaks and shaping the signal into something like a sqaure wave.
The Fuzzy Mk. 1, as my debut on the scene of DIY guitar pedals, is a simple fuzz pedal.
It offers a basic fuzzing effect and a buffered bypass controlled by a toggle switch.
The intensity of the fuzzing can be controlled (down to being outright disabled) by a knob.
![Schematic of fuzz pedal circuit](images/schematic.svg)
## Circuit Operation
### Core Effect
The core fuzzing effect is implemented with a single op-amp circuit, which combines inverting amplification with limiting.

While I could have implemented the clipping by simply amplifying the signal past the supply voltages, there are a few issues with this.
For one, this would leave the signal at or near the supply voltages, which means it would need to be attenuated before passing it to the next pedal in the effect chain.
This would necessitate a second amplifier stage, so I would either have to give up the buffered bypass, or include a second op-amp on the board, which would increase costs.
Not only that, but variably attenuating a signal back to its original amplitude is a design challenge I did not want to have to solve.
Another issue is that some op-amps have different capabilities between the two supply rails: they can clip to the negative closer than they can clip to the positive.
This would result in an uneven clipping.
But most of all, I consider it poor design practice to deliberately exploit maximum ratings to achieve an effect.

The inverting amplification is implemented with two fixed resistors (R4, R5) and a potentiometer (RV1).
The maximum gain is -11 V/V, and by including R5 in series with the potentiometer, the minimum gain is -1 V/V (instead of 0).
The amplified signal is clipped at 3V and 6V, resulting in a maximum peak-to-peak amplitude of 3V.
This is one-third of the supply voltage of 9V, allowing for considerable headroom without overly attenuating the signal.
### Bypass Toggle
Most guitar pedals include a switch to toggle bypassing the effect, which means that the signal is passed directly to the next pedal in the chain.
In the Fuzzy Mk. 1, this toggle functionality is implemented with an analog 2:1 multiplexer, an electronic toggle latch, and a momentary pushbutton.
1. The momentary pushbutton, the only moving component on the board, acts as the user interface. It is only active as long as it is held down.
2. The electronic toggle latch, implemented with a 555 timer in bistable operation, converts the momentary activation to toggle activation.
3. The output of the toggle latch controls a low-side switch, implemented with a single NPN transistor, which pulls down the multiplexer's control pin.
4. The ADG419 2:1 analog multiplexer routes one of the two different signals to the output.
## Component Selection
For my first pedal, I elected to stick with simple through-hole components, including an op-amp for the core functionality of the pedal.
### NE5532 Operational Amplifier
The NE5532 op-amp is a classic op-amp that has been popular with audio enthusiasts for a few reasons.
#### Gain-Bandwidth Product
For one, TI's rendition of the NE5532 boasts a gain-bandwidth product of 12MHz.
In order for an amplifier to be useful for boosting audio signals, it needs to be able to handle the whole range of human hearing, which is about 20Hz to 20kHz.
By dividing the gain-bandwidth product by the upper end of human hearing, you get the maximum useful gain of an op-amp for audio signals.
In the case of the NE5532, you get a maximum useful gain of 600 V/V, or about 55.6dB.
In comparison, the antique LM741 has a gain-bandwidth product of 1.5MHz, which means it can only boost audio signals up to a gain of 75 V/V, or about 37.5dB.
#### Slew Rate
Another important feature of the NE5532 is a slew rate of 9 V/μS, or 5 V/μS for TI's version.
The slew rate of an op-amp is the rate at which the voltage on its output pin can change.
In order for an amplifier to be useful for boosting audio signals, it needs to be able to slew fast enough to keep up with the signal.
To find out if the amplifier can do so, take the derivative of a sine (or cosine) wave at 20kHz with the desired peak-to-peak amplitude (actual voltage level, not gain), and find the derivative's maximum.
We only need to check the highest frequency because if the amplifier can slew fast enough for the highest frequency, it can slew fast enough for lower frequencies.
For example, if your amplifier must produce a 10V peak-to-peak signal, then the highest frequency can be represented with the equation y(t) = 20sin(20000\*2pi\*t).
The derivative of y(t) is y'(t) = 20cos(20000\*2pi\*t)\*20000\*2pi, via the Chain Rule.
We are only concerned with the maximum of the derivative, which of sine and cosine, regardless of the parameter, is 1.
Therefore, the maximum is simply the constant coefficient, which in this case is 20\*20000\*2pi, or about 2.51\*10^6.
To adjust the value to V/μS, divide that value by 10^6, which in this case results in about 2.51 V/μS.
For the fuzz pedal, or any standard guitar pedal, the supply voltage--thus the maximum peak-to-peak amplitude--is 9V.
Therefore, the necessary slew rate is 1.13 V/μS, well under the maximum slew rate of the NE5532.
In comparison, the slew rate of the LM741 is 0.5 V/μS, which is not enough to amplify the whole human hearing range up to 9V peak-to-peak.
By the way, I believe that the slew rate is the reason why the cut-off peaks are not perfectly squared off but somewhat rounded, as seen in the LTspice simulation.
### ADG419 Analog Multiplexer
There's not much to say about the component itself.
I only selected it because it was what Digikey had for 2:1 analog multiplexers in a through-hole package.
What matters is why I used an electronic switch at all, as opposed to just a mechanical switch.

Mechanical switches have a few downsides, and the upsides are not relevant to small-signal applications, such as a guitar pedal.
Mechanical switches undergo strain and are more prone to physically breaking compared to electronic switches.
By using an electronic switch, the signal path does not have to routed through the mechanical switches, which has constraints on where it can be placed on the board (the mechanical switch is also the user interface); in other words, signal routing and user interface are separated from each other.
Finally, the electronic switch provides programmability.
With just a mechanical switch, you are limited to how the switch is physically designed.
For example, a momentary switch can only ever be momentary, unless you tape it down.
But with the electronic switch, you can build a circuit around it to interact with it however you want.
In this circuit, as seen below in the section on the NE555 timer, I built a toggle latch circuit so that the user can toggle between fuzz and bypass.
### NE555 Timer
The 555 timer, as it is colloquially referred to, is a versatile IC that is most commonly used for analog timing applications.
In this circuit, the timer runs in bistable operation, which means that it has two stable states: on and off.
More specifically, the timer operates as a toggle latch, so that the momentary pushbutton becomes a toggle pushbutton.
This toggle functionality could have been implemented in a variety of ways, but the 555 timer minimizes the number of components needed to do so.
Moreover, it is a very common part and easily replaced.
This let me order TI's NE555 in bulk so that I can use it in future projects, thus cutting down on costs.
### Small Components
The small components consist of a few common resistors and capacitors, as well as a couple diodes and a single discrete transistor.
For the resistors, I used what I had lying around, because whatever came in my Arduino kit is probably common and cheap enough to easily supply and replace.
These consist of blue metal-film resistors, which while being harder to read, are better because they lack the inductance of wire-wound resistors.
Most of the capacitors are ceramic, because they are not polarized and generally offer better characteristics; anything 10μF or above are aluminum electrolytic, mainly for bypass.

The diodes are 1N4148 diodes.
Unlike the resistors, I did not use what I had lying around, because what I have are 1N4007 diodes, which are for rectification and not for signal processing.
Conversely, 1N4148 diodes are better suited for signal processing, with faster switching and a lower forward voltage.
On the other hand, 1N4148 diodes are similarly cheap, cutting down on costs.

There is a single discrete transistor in the circuit, acting as a low-side switch for the ADG419 control pin.
When brought into saturation, it brings the control pin of the ADG419 down to ground, switching the output from the buffer to the fuzzer.
The schematic shows a 2N3904 transistor, which is a commonplace NPN BJT, because it's what KiCAD had in its standard parts library.
However, I ended up just soldering one of my 2N2222 transistors into the board, because that's what I had on hand, and both could work easily.
Really, any low-power NPN or NMOS transistor could work, though for MOS, the base/gate resistor (R11) is rendered unnecessary, because the MOSFET gate terminal draws virtually no current.
But again, 2N2222 is what I have on hand, and using a BJT and a resistor I already own is cheaper than buying a MOSFET.

The board also includes a 1MΩ potentiometer, for controlling the intensity of the fuzzing effect.
Realistically I could have used a 100kΩ or even a 10kΩ pot, but the large resistance value is a vestige from when the DC bias was on the signal input and not the noninverting op-amp input.
In that case, I had to ensure the voltage divider providing the DC bias was stiff compared to the feedback network, so the feedback resistors (including the pot) had to be relatively large.
## Simulations
This repository includes two LTspice simulations: one for the fuzzing effect and switching the output, one for the 555-driven toggle latch.
Not being able to afford laboratory equipment, this was the next best thing.
LTspice even came with ADG419 as part of its standard library of parts, since both LTspice and ADG419 are made by Analog Devices.
## Design Challenges
There were a few challenges involved with the design of this pedal:
 - Guitar pedals use a single supply rail (9VDC) instead of split supplies, meaning the input had to be DC biased.
 - Guitar pedal design standards have unusual quirks, which I had to keep in mind, such as the negative-center configuration of the power jacks.
 - User-facing components (buttons, knobs, connectors) dominated the layout, which was especially critical when the signal is routed through the component (i.e. through the potentiometer).
 - I committed to a two-layer board layout with through-hole components and a ground plane, making routing tricky in some places.
## Future Revisions
There are several improvements I can make for future revisions of this project:
 - Update some older components, such as replacing the NE5532 with TI's TLV9362 op-amp.
 - Use surface-mount devices instead of through-hole devices, thereby saving board space and reducing costs.
 - Replace the ADG419 with a less expensive alternative, as it is the most costly electronic component on the board.
 - Implement safety features, such as overcurrent protection and transient voltage suppression.
## References
- https://www.allaboutcircuits.com/technical-articles/an-op-amp-limiter-how-to-limit-the-amplitude-of-amplified-signals/
- https://youtu.be/dhzswWLUoG0?si=y4rR1ria-jn_BZjD