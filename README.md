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

The specifications will hold cross process corners - TT,SS,FF,FS,SF and Temperatures ranging (-40,125)

# Hand calculations & Design assumptions
2-stage Opamp will have 2 pole's which can cause a non-stable amplifier (PM < 45 [deg]), therefore we will use pole-splitting compensation by adding a Capacitor Cc between 1st and 2nd stage.
The compensation capacitor value can be: $C_c \approx 0.25*C_L$, which will decrease the dominant pole (P1) and increase the non-dominant pole (P2) but will introduce a **RHP ZERO!!!** (Z) which acts as a pole and can reduce PM.

There are 2 ways to compensate the Zero: 
* move Zero to high frequencies -> high power design
* move Z to LHP -> can introduce a Pole-Zero doublet (which will affect ringing)
We will show both techniques.

**Pole Zero analysis**

$GBW = g_{m1}/C_c$

$P_1 = BW = f_d = 1/[(r_{o2}||r_{o4})C_c]$

$P_2 = f_{nd} = g_{m6}/C_L$

$Z =  g_{m6}/C_c$

For the first compensation method: $Z > 10*GBW$

For the second compensation method: $Z \approx P_2$

**Parameters analysis**

$SR = I_{tail}/C_c$

Open loop Gain: $A_v = A_{v1}*A_{v2} =  g_{m2}*r_{o2}||r_{o4}*g_{m6}*r_{o6}||r_{o5}$ (assuming M1=M2, M3=M4)

We will calculate $g_{m1}$ from Settling time ($T_{slew} + T_{sm,sig}$) equations (see attched excel)

We will target for $V_{dsat}$ of 0.15-0.2V for the current mirrors M3,M4,M5,M7 and $V_{dsat}$ of 0.05-0.1V for the gm transistors M1,M2,M6

**Beta multiplier**

We will use a basic Beta-multiplier circuit with start-up circuit to assure circuit will wake-up.
The circuit will produce a $20[uA]$ bias current, according to the following equation - $I_{out} = (2/U_nC_{ox}R_s^2)*(1-1/\sqrt(K))^2$

# Design & Simulations
## Cc Miller compensation:
Taken from Willy Sansen book - Analog Design Essensials - 

<img src="https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/9eefba00-3ce6-4354-a655-f47b7b170388" align="middle" width="500" height="500"  alt="Image Alt Text" />

Because ICMR- is very low, we are using a differential pair PMOS based as the first stage, following by a NMOS Common Source as the 2nd stage.

Final design, where $C_c = 4[pF]$ - 
![image](https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/d39115f1-f178-4788-87af-ae4e370e25f4)

Looking at STB simulation results and plotting Bode plot before (Red and Yellow) and after Cc Miller (Green and light Blue), we can see that the Cc miller CAP increased PM by splitting the poles - $f_d$ is decreased to $\approx 1k$ and $f_nd$ increased to $\approx 40[MHz]$ - 
![image](https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/67431c4a-852f-43b3-9f12-bd20c447b26a)

PM increased from 27[deg] to 75[deg] ! 

We can also see we have no RHP zero (probably moved to higher freqencies), it is because we kept $Z > 10GBW$ by keeping $gm_1 \approx 0.2mS > 10*gm_6 \approx 2mS$ by the penalty of burning more power in 2nd stage 

