# 2-stage-OTA-buffer-w-Beta-multiplier
In this project we will show the design of a 2-stage Operational Transconductance Amplifier (OTA) in unity gain closed loop configuration ("Buffer").

We will also provide bias current with a Beta multiplier circuit, and show 2 pole-splitting compensation techniques -
* Cc Miller compensation - for high power consumption
* Cc+Rc PVT tracking compensation - for low power consumption

We will use CMOS general pdk 90nm (gpdk90) technology by Cadence.

**Specifications**
* $Gain > 70dB$
* $PM > 60 [deg]$
* $ICMR- = 0.4[V]$
* $ICMR+ = 1.4[V]$
* Settling time: $T_s < 1[us]$
* Step input: (0.5, 1.5)
* Voltage accuracy: $V_{acc} = 1[mV]$
* Slew Rate (SR) > 4[V/us] 
* $V_{AA} = 2.5[V]$
* $C_{L} = 20[pF]$

# Hand calculations & Design assumptions
2-stage Opamp will have 2 pole's which can cause a non-stable amplifier (PM < 45 [deg]), therefore we will use pole-splitting compensation by adding a Capacitor Cc between 1st and 2nd stage.
The compensation capacitor value can be: $C_c \approx 0.25*C_L$, which will decrease the dominant pole and increase the non-dominant pole but will introduce a **RHP ZERO!!!** which acts as a pole and can reduce PM.

There are 2 ways to compensate the Zero: 
* move Zero to high frequencies -> high power design
* move Z to LHP -> can introduce a Pole-Zero doublet (which will affect ringing)
We will show both techniques.

**Pole Zero analysis**

$GBW = \frac{g_{m1}}{Cc}$

$P_1 = \frac{1}{(r_{o2}||r{o4})A_{v2}C_c}$

$P_2 = \frac{g_{m6}}{C_L}$

$Z =  \frac{g_{m6}}{C_c}$

For the first compensation method: $Z > 10*GBW$

For the second compensation method: $Z \approx P_2$

**Parameters analysis**

$SR = I_{tail}/C_c$

Open loop Gain: $A_v = g_{m2}*r_{o2}||r_{o4}*g_{m6}*r_{o6}||r_{o5}$ (assuming M1=M2, M3=M4)

We will calculate $g_{m1}$ from Settling time ($T_{slew} + T_{sm,sig}$) equations (see attched excel)

We will target for $V_{dsat}$ of 0.15-0.2V for the current mirrors M3,M4,M5,M7 and $V_{dsat}$ of 0.05-0.1V for the gm transistors M1,M2,M6

# Design & Simulations
## Cc Miller compensation:
Taken from Willy Sansen book - Analog Design Essensials - 

<img src="https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/9eefba00-3ce6-4354-a655-f47b7b170388" align="middle" width="500" height="500"  alt="Image Alt Text" />

