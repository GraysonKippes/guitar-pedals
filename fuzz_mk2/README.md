# Fuzzy Mk. 2
The Fuzzy Mk. 2 is an analog fuzz-style distortion pedal with two-band tone control.

## Circuit Overview
The effect path consists of two stages: the fuzz distortion, then the tone control.
The fuzz distortion is the main part of the circuit, and the tone control is an add-on inspired by the Big Muff Pi requested by my client.
I upcharged him (and every other buyer) for it.

### Fuzz Distortion
Fuzz distortion works by heavily amplifying a signal past the output range of the amplifier, resulting in a heavily clipped waveform that resembles a square wave.
Like the Mk. 1, I implement this clipping not with overdriving an amplifier, but with an op-amp limiter circuit.
This means I control the limit level in the signal feedback path, rather than by adjusting the voltage delivered to the op-amp's supply pins.
Op-amps often have assymetrical headroom under the positive rail and over the negative rail; i.e. output can get closer to the negative rail than to the positive rail.
This would result in an assymetrical clipping, which, while probably an interesting distortion effect, is not what I am going for.
(At any rate, I could implement assymetrical clipping in the signal feedback path if I wanted to.)

Also, controlling the limit level through the power rails would most likely amount to powering the op-amp through a voltage divider, and the op-amp itself would end up loading the voltage divider when it draws current, changing the divider ratio.
On top of that, using the power rails to control the signal in such a way would leak any ripple in the power supply into the signal, which is incredibly undesirable.

As a side note, I believe that one could implement "softer" distortions (such as overdrive) by increasing the resistances of the resistors in the limiter voltage divider, relative to the resistors in the amplifier voltage divider.
This would cause the limiter to "sag" and not clip the signal as hard.

### Tone Control
The tone control works by splitting the signal path into two paths: one that is fed into a low-pass filter, and one that is fed into a high-pass filter.
The two paths are then added back together.

More specifically, my tone control is essentially the Big Muff Pi (by Electro-Harmonix) tone control, with two crucial modifications.
1. The filters and the adder circuits were converted from one big passive network to three op-amp circuits, effectively buffering each stage.
2. The filters were designed to share the same cut-off frequency, rather than have two separate frequencies.

By converting the passive network into distinct active stages, they no longer sag each other down.
Also, by making the tone control stages active, I can further boost the signal if needed and buffer the output, not having a high impedance output.

The original Big Muff Pi pedal has a different cut-off frequency for each filter, with a lower cut-off for the low-pass filter and a higher cut-off for the high-pass filter.
When the two filtered paths are added back together, a gap is left behind in the frequency response.
This is the "mid-range scoop" for which this pedal is infamous; guitarists have agonized over how to compensate for this, when I simply restore that band of frequencies.

The adder stage of the tone control consists of nothing more than a potentiometer (which connects the outputs of the filters together) with the center tap buffered by an op-amp.
This allows the user to set a ratio with which the low-pass and high-pass lines are mixed.
For example, setting the pot wiper three-quarters along the way to the low-pass end means that the output will receive 75% low-pass and 25% high-pass.

My main goal with the tone control was to have it so that the frequency response would be totally flat when set to 50/50 mix; that is, the signal would sound the same, as though there was no tone control at all.
matching the cut-off frequencies of the filters ensures this, and this can be proven mathematically by adding the frequency responses of the two filters:

$$ \frac{1}{1 + sRC} + \frac{sRC}{1 + sRC} = \frac{1 + sRC}{1 + sRC} = 1 $$

This result is confirmed with an LTspice simulation of the tone control circuit.

## Component Selection
In my opinion, the only components worth discussing in this circuit are the op-amps; no need to discuss the resistors and capacitors.
I originally wanted to pick out nice op-amps from the Burr-Brown line, and I spent a lot of time sifting through the parts.
I settled on an OPA1641 for the buffer input and an OPA1679 for the signal processing.
However, by the time I got around to ordering the parts, the OPA1679 went out of stock.
(This has happened before too, with an STM32H7A3, which is why that project is on hiatus).
So I bought a TL074 instead, which is thankfully pin-compatible.

All the noiselessness of the newer op-amps is pretty much pointless anyways, given the fact that guitar pedal setups are riff with stray inductances and ground loops.
So for this reason (and a few others), I will probably just stick to the TL07x and NE5532 op-amps going forward.

I picked the OPA1641 because it is a JFET input and thus has much higher input impedance and can handle higher impedance sources.
According the Google, JFET inputs are supposed to have this rich, warm tone to them that guitarists like.
It's probably nonsense, but when in Rome, do as the Romans do.

I picked a quad op-amp package for the signal processing to fit it all into one neat package, but it ended up being more a hassle to route to be honest.
The main benefit I see of quad op-amps is that they seem to all or mostly use the same standard pin-out, whereas single- and dual- packages have different pin-outs.
I have read before that quad op-amps have all these issues, including increased crosstalk noise and higher power consumption; but to be honest, other than the more restricted routing, I doubt that these supposed issues are real, or if they are real, actually matter.
Again, the noise from patch cables and ground loop hums dominates whatever noise there is or isn't in the op-amps.

## Comparison to Fuzzy Mk. 1
In all honesty, the first fuzz pedal was completely overengineered with the analog mux and the 555 timer used as a bistable latch.
Getting rid of those massively simplified the circuit design and freed up room for other circuitry (namely the tone control).
At any rate, the 555-based toggle functionality was really buggy and just a nuisance overall.
Probably the biggest embarrassment in my opinion was using the 555 timer output to control a transistor switch that in turn controlled the mux, which was utterly pointless.

I still believe that an electronic switch (controlled by a mechanical switch) is superior to a purely mechanical switch for a few reasons.
1. Switch bouncing can be eliminated by placing a low-pass filter on the base/gate of the electronic switch.
2. The signal path does not have to be run all the way back to the footswitch (optimizing PCB trace length is pointless for noise, but it would make routing easier).
3. The switch could be controlled however I want, and I am not just limited to the mechanical switch functionality.
That being said, I would rather implement the electronic switching with discrete transistors than spend $6 on a single mux IC again, especially since switches are needed on both the input and output, doubling the cost.

## Future Revisions
There are a few revisions I would like to implement for Fuzzy Mk. 3, should I ever build it.
 - Design the board such that it can fit in a standard metal stompbox, so that I don't have to have a custom 3D printed case.
 - Put the footswitch on the lower end of the board, underneath the potentiometers, so that the guitarist doesn't accidentally stomp on the pot knobs.
 - Implement electronic switches using discrete transistors, as described above.
 - Just use two dual-channel TL072s or NE5532s for the effect path (and a TL071 for the input buffer).
 - Include some stylish graphics on the board for the sake of posterity.

 - 