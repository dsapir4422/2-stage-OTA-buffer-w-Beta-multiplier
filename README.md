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

The specifications will hold cross PVT corners: process - TT,SS,FF,FS,SF, Temperatures ranging (-40,125) and +/- 10% supply (2.4,2.5,2.6), total 54 corners -

<img width="250" alt="image" src="https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/6b650ba0-0211-48cf-8e6a-2f956469dc7b">


# Hand calculations & Design assumptions
2-stage Opamp will have 2 pole's which can cause a non-stable amplifier (PM < 45 [deg]), therefore we will use pole-splitting compensation by adding a Capacitor Cc between 1st and 2nd stage.
The compensation capacitor value can be: $C_c \approx 0.25*C_L$, which will decrease the dominant pole (P1) and increase the non-dominant pole (P2) but will introduce a **RHP ZERO!!!** (Z) which acts as a pole and can reduce PM.

There are 2 ways to compensate the Zero: 
* move Z to high frequencies -> high power design
* move Z to LHP -> can introduce a Pole-Zero doublet (which will affect ringing)

We will show both techniques.

**Pole Zero analysis**

$GBW = g_{m1}/C_c$

$P_1 = BW = f_d = 1/[(r_{o2}||r_{o4})C_c]$

$P_2 = f_{nd} = g_{m6}/C_L$

$Z =  g_{m6}/C_c$

For the first compensation method: $Z > 10*GBW$

For the second compensation method: $Z \approx 2.2P_2$

**Parameters analysis**

$SR = I_{tail}/C_c$

Open loop Gain: $A_v = A_{v1}*A_{v2} =  g_{m2}*r_{o2}||r_{o4}*g_{m6}*r_{o6}||r_{o5}$ (assuming M1=M2, M3=M4)

We will calculate $g_{m1}$ from Settling time ($T_{slew} + T_{sm,sig}$) equations (see attched excel)

We will target for $V_{dsat}$ of 0.15-0.2V for the current mirrors M3,M4,M5,M7 and $V_{dsat}$ of 0.05-0.1V for the gm transistors M1,M2,M6

**Beta multiplier**

We will use a basic Beta-multiplier circuit with start-up circuit to assure circuit will wake-up.
The circuit will produce a $20[uA]$ bias current, according to the following equation - $I_{out} = (2/U_nC_{ox}R_s^2)*(1-1/\sqrt(K))^2$

**Rz PVT tracking**

As mentioned earlier, adding a C_c capacitor introduce a Zero which we would like to cancel. If we keep the Zero close to P2 it will behave as LHP Zero and the pole and zero will cancel each other out. In order to do that we will design - $R_z = (1/g_{m6})*(C_L + C_c / C_c)$ (gm6 - 2nd stage CS). For good design over corners, we will design $R_z$ as a transistor in low triode region, and add a Vgs divider which will track voltage changes across corners (PVT tracking)

# Design & Simulations
## Cc Miller compensation:
Taken from Willy Sansen book - Analog Design Essensials - 

<img src="https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/9eefba00-3ce6-4354-a655-f47b7b170388" align="middle" width="500" height="500"  alt="Image Alt Text" />

Because ICMR- is very low, we are using a differential pair PMOS based as the first stage, following by a NMOS Common Source as the 2nd stage.

DC operation point simulation, where $C_c = 4[pF]$ - 
![image](https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/d39115f1-f178-4788-87af-ae4e370e25f4)

Looking at STB simulation results and plotting Bode plot before (Red and Yellow) and after Cc Miller (Green and light Blue), we can see that the Cc miller capacitor increased PM by splitting the poles - $f_d$ is decreased to $\approx 1[KHz]$ and $f_nd$ increased to $\approx 40[MHz]$ - 
![image](https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/67431c4a-852f-43b3-9f12-bd20c447b26a)

PM increased from 27[deg] to 75[deg] ! 

We can also see we have no RHP zero (probably moved to higher freqencies), it is because we kept $Z > 10GBW$ by keeping $gm_1 \approx 0.2mS > 10*gm_6 \approx 2mS$ by the penalty of burning more power in 2nd stage.

Simulating over corners - 

<img width="600" alt="image" src="https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/479e1110-a5ea-4e7d-bf71-bb850f10f568">

We will replace the ideal current source with the Beta multiplier circuit - 

![image](https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/6cb5a597-aabf-40d9-a2ef-e814df845785")


M0,M1,M2 assemble a start-up circuit, M14,M18-M20 + R1 are a self-biased Beta-multiplier which will provide the OTA a 20[uA] bias current.
Let's re-simulate with Beta multiplier + 2-stage OTA + Cc Miller capacitor -

<img width="500" alt="image" src="https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/01e2e498-3386-4aba-bdb4-25cda932eb4e">

Simulation results are still in spec, but STD increased as we replaced ideal components (current source) with Beta-multiplier

## Cc+Rc PVT tracking compensation:
We will now show the same design but with a different compensation technique - we will add a resistor in series with Cc, so now we can reduce 2nd stage current as we don't need $Z > 10GBW$. also, as long as we keep Rz Zero close to $f_{nd}$, the LHP Zero and Pole will cancel each other ! the disadvantage if we make a poor design is a Pole-Zero doublet which will cause ringing in output signal. 
We will also implement Rc resistor with a PVT tracking circuit to be able to track the resistance over corners. 

DC operation point simulation, $R_c = 5.6[KOhm]$ - 

![image](https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/0bd3a8d3-0a57-40cc-81a3-8e30299defa5)

Looking at STB simulation results and plotting Bode plot before (Yellow) and after (Red), we can see PM increased from 35[deg] to 84[deg] !
![image](https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/72d89a24-8b8d-448f-b13a-b3ed4680be38)

Replacing ideal component - current source with Beta multiplier circuit and Resistor with PVT tracking circuit
Final design - 
![image](https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/e8614705-ea3c-4131-91ae-e8cd24f5e7e4)
![image](https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/77cda200-fc90-43fe-8eb4-e311a7a3c39c)

Simulation results over corners -

<img width="500" alt="image" src="https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/25bda51e-2f30-476a-a62c-169d919c8ede">


Full circuit -
![image](https://github.com/dsapir4422/2-stage-OTA-buffer-w-Beta-multiplier/assets/87266625/62da1767-9ccb-4387-8d84-922ab4ff5d2e)



