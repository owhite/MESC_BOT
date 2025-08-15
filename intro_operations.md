# MESC_Firmware
MESC is a project for embedded BLDC FOC, serving a number of purposes
1) Easy to follow and learn FOC
2) Easy to port to other platforms
3) High performance motor control offering all the FOC goodies: Sensorless, HFI, Encoder, Hall, (and combinations of), Field weakening, MTPA, Torque, Speed and Duty control.
4) Permissive licensing making commercial use easy (Additional conditions attached to integration into other open source projects).


MESC was written by David Molony and his github repository is here: https://github.com/davidmolony/MESC_Firmware


With thanks to all that have helped in the creation of MESC.


The author and contributors of MESC wish you “Happy spinning!”. That is our motto. 


# Targets/Hardware
MESC runs primarily on any STM32 target with an FPU. Tested with the targets in the repo, but easily portable to any other STM. Compatibility with other MCU brands TBC.


The reference hardware is now the [MP2 ESC](https://github.com/badgineer/MP2-ESC), since it allows testing with many targets by simply swapping the MCU pill. It also offers adequate performance for most light EV applications (high power scooters and ~10kW E-motorbikes.


All STM32F405RG hardware (AKA VESC compatible hardware e.g. Trampa, SHUL, Triforce, FSESC and many others) compatible with MESC_firmware.




# Introduction


## Foreword:
This project is new as of 28/06/2020, and is the work of David Molony, experienced mechanical and electrical engineer, and software learner.   
Contributions from others welcome, but 
1) The project is not intended to be an all inclusive do everything like VESC. This project is intended to be FOC that "just works" and is trivial to understand, port and build applications with.
2) The project will remain in its entirety BSD 3 clause, MIT or other equivalent entirely permissively licenced.
3) As the project matures, while the code style remains nothing special, there are increasing numbers of originally developed and both effective and easy to implement techniques that are offered permissively. Borrowing sections of code for other projects is allowed, but without exception, if you borrow the code, even if you rename variables or break it up/relocate into various subroutines, you MUST credit the origin, and maintain the permissive BSD licencing inline if necessary. Failure to do so means you grant a perpetual permissive licence to your project and lose your rights to using this code permissively. This statement is in response to a pernicious yet ubiquitous habit of open source projects ripping out permissive licencing or taking public domain code and re-issuing under copyleft terms. The origin of this ire was searching for python source and finding the entire cpython project online (since removed) with all trace of the BSD/PSF licence removed and replaced with GPL.


**Thanks to contributors, especially:**


* c0d3b453 for large amounts of helpwith C and teaching, motor, speed temp and other profiles
* Salavat for initial STM32 setup and teaching
* Elwin (offline) for testing, motor control idea bouncing and assistance with current controllers
* Jens (Netzpfuscher) for RTOS, Terminal and variable save v2, tidying up and contributions to SinLUT
* OWhite for creating the Github Pages


## Licence
This project will initially (and perhaps perpetually) contain a lot of firmware licenced under the STM Cube licence, BSD 3 clause https://opensource.org/licenses/BSD-3-Clause .   
The rest of the custom code is intended to be contained primarily in the MESC files, also BSD 3 Clause.
It is requested that if you borrow, port, refactor parts into your own code... etc this firmware, you may let the project owner know, But you are not compelled to. 
It would also be nice if concise and focused improvements were contributed back, but again, you are not compelled to.


You must retain credit for the code origin in your source, even on small segments borrowed or reimplemented. This is compelled.


If this code is borrowed in part or whole for use and published in GPLV2 V3... projects, you must ensure that the origin of the code is clear and that it is clear that there is no GPL obligation of the relevant sections to avoid any ambiguity or claims of GPL infringement.
Bugfixes or "improvements on the theme" made by GPL projects should be allowed to be back ported to this project as BSD-3-Clause. 
None of the above statements shall be construed as modifying the licence of another project's original work. The intent is purely to avoid a situation where use in multiple projects causes concern about integration into proprietary code, or further development around similar themes might give rise to infringement concerns.


The above is to protect the ability/right for this code to be used in closed source or proprietary systems; as segments already have been.
For commercial, closed source and permissively licenced projects, the code can be used as BSD-3-Clause. The intention of this project is to be useful to whoever wants to use, learn, build, integrate, sell or even lock down a version for regulatory compliance purposes.


## Noteable features
* Simple and robust sensorless observer 
* PWM-Synchronous Encoder readings
* HFI using d-q coupled current or 45 degree injection
* Dead time compensation and characterisation
* 100% modulation techniques
* Field weakening and MTPA
* Fast fault shutdown from three possible sources - BRK input (<1clock cycle MCU propogation), ADC Watchdog (optional, runs at PWM frequency by default and guards Out of range) and software comparison.
* Parameter (Rs, Ld, Lq, flux linkage detection, HFI thresholds)
* Operation up to 100kHz+ when running V0 only (HFI, encoder, logging disabled)
* Operation up to ~70kHz PWM (140kHz V0V7 frequency) with F405 MCU, and some options disabled (e.g. SPI encoder). Operation to about 40kHz with F401 MCU and about 35kHz with F303. Stable to <2.5kHz PWM frequency, though this is definitely not advised for most applications.
* Most hardware cannot cope with current measurements above about 60kHz, noise becomes prohibitive.
* Easy porting to any STM32 with a floating point unit and timer1
* Probably easily portable to any other MCU with FPU, 3 phase timer and a 1MHz+ ADC


# Theory and Operation


## Hardware:
Any STM32 based 3 phase system using Timer1 for PWM High and Low and 3 phase current plus bus voltage measurement. Canonical hardware is F303, but better results can be made with external amplification so the F303 is no longer preferred.
Preferable to have phase voltage sensors for restart while spinning and tracking without modulation (gives reduced drag and better efficiency)
Specifically intended for the MESC_FOC_ESC and MP2 hardware, but also running on F405, F401, L431 and F411 targets.  


## Forenote:
After the explanation in the PWM section, it will be assumed that timer1 refers to any timer used for the PWM generation, which is propogated as mtr[n]->mtimer after being introduced as a handle in the PWM ISR. 
Multi motor is a new concept originating December 2022.
PWM frequency and V0V7 frequency are distinct, and there is latent confusion since some codebases and manufacturers like to cite ambiguously "switching frequency" since it gives the impression the system is capable of double the frequency it really is.
When choosing a PWM frequency when coming from another motor control system, be aware the convention used. VESC, ASI some BLDC controllers and probably others cite switching frequency. MESC uses PWM frequency so you may need to choose a value half what you used before, e.g. VESC 30kHz is equivalent to MESC 15kHz.




### Center Aligned PWM
//ToDo, add pictures and exlpanation of PWM center aligned generation for 3 phase currents. Anyone already reading this... There are quite a few resources that can explain this on the web already.


### PWM
Timer1 (or Timer 8 if using dual motor) set up to generate complimentary centre aligned PWM with dead time, with frequency configurable using #define PWM_FREQUENCY 25000 in your hardware apecific header.


These timers are assigned an alias specific to the motor instance such that mtr[0]->mtimer = htim1 and mtr[1]->mtimer = htim8. It is also possible to use htim20 on large STM devices.


The center aligned PWM counts (mtimer->Instance->CNT) up from 0 to ARR (Auto Reload), where the direction is changed and it starts to count down. When the count is below the CCR (Compare Register) the timer outputs HIGH to the high side MOS and LOW to the low side MOS, and the phase is connected to VBat. When it is above the CCR, the two outputs swap, and the phase is connected to ground.
This means that a higher CCR gives a larger pulse of bus voltage to the motor.
 
Space vector modulation variant "mid point clamp" primarily used; bottom clamp implementation exists, and is used for overmodulation, but results in less good FET currnt sharing.
Centring at 50% duty cycle has a few advantages - it allows recirculation through the high side FETs as well as the low side, which evens out the load on them, and cancels most offsets. 
The SVM is implemented as the true inverse clarke transform (2 phase to 3 phase) followed by selection of the highest and lowest voltage phases to generate a mid point. This mid point is then fixed to mtimer->ARR/2, effectively allowing the unipolar DC link to generate AC voltages.


For higher modulation indices (typically >95%), bottom clamp implementation can be used for FOC control. It is a bit noisier, less FET sharing, but allows for longer current sampling periods, and therefore higher modulation. 
SVPWM mid point clamp assumes the circle limiter clips the Vd and Vq such that the inverse transform is limited to what will fit within the bus voltage.


There is no hard limit on ERPM (= 60 x eHz), but eventually it trips. Has worked to over 300kerpm on F405 targets with a GaN power stage. This is uselessly high in practice.
Most motors cannot cope with this speed due to stator eddie and hysteresis losses. Typical 0.2mm laminations become very inefficient at around 1kHz.
The limit of proper commutation is roughly 20PWM periods per eHz, so at 20kHz PWM, it is possible to run to 1000eHz with good performance. This can be increased slightly by enabling interpolation (#define INTERPOLATE_V7_ANGLE)


#### PWM ISR (Interrupt Service Routine, aka IRQ) generation
The mtimer generates the interrupts for the FOC, with interrupts enabled at over and underflow of the timer (when CNT = 0 and ARR). The configured ISR this enters through is TIM1_UP_TIM10 at vector address 0x0000 00A4 on STM32F405. ST remap this to the "stm32f4xx_it.c" where it enters the function: 
void TIM1_UP_TIM10_IRQHandler(void)
Within this ST calllback, MESC calls 
MESC_PWM_IRQ_handler(&mtr[n]) where n would typically be 0 for the timer1 callback, 1 for timer8 callback and 2 for timer20 if used. This function is implemented in MESC_Common in MESCfoc.c


From here on, the code is largely generic to MESC (exception being the getRawADC(_motor) function which has to return the specific ADC readings mapping the ADC to the MESC RAW data.


Owing to the number of clock cycles per PWM period for the MCU to complete math (e.g. for STM32F303 72MHzcore/25KHzpwm/2 = 1440 clock cycles) math occurring in the interrupt must be fast.
Firmware will be targetting <1000 clock cycles per Fastloop and <<1000 cycles per hyperloop to enable high frequency operation. Therefore, many functions and precalculations are pushed into the slowloop.


The MESC_PWM_IRQ_handler calls fastLoop(_motor) when the timer is counting down, and hyperloop(_motor) when the timer is counting up (note that _motor represents mtr[0], mtr[1], mtr[2]... depending on what handle was passed in the PWM ISR function!)
fastLoop contains the FOC control, the feedback between current measurement and voltage output and the sensorless observer.
hyperLoop contains the HFI injection function, angle interpolation and when used, synchronous encoder readings. 
Both contain a call to the SVPWM function writePWM(_motor) so that the PWM can be updated 2x per PWM period.


### ADC
ADC conversions are triggered (preferably, where available) by mtimer CCR4 on TRGO. ADCs 1,2,3 are used to get fully synchronous current readings on F303 and F405, but on other targets, or with multiple motors, they sometimes must be sequential. This does not in practice result in a noticeable loss of performance. Vbus is read immediately after current reading as the next most critical parameter.


Within the fastLoop, which occurs at mtimer top shortly after the ADC conversion has been initiated, a call is made to ADCConversion(_motor) which calls getRawADC(_motor) which is a function specific to the hardware target, and must be implemented in the user's MESChw_setup.c file if different to existing targets. Once the raw values are attained, the fastLoop procedes with no hardware specific call.


ADC interrupt is now reserved for overcurrent protection events with the analog watchdog and it is recommended to set the threshold to a range the current sensors can reliably reach; typically 10-50 counts for opamp and perhaps up to 400 counts for inductive sensors. These thresholds can be set in CUBEMX for new hardware or can be written directly to the hadc1.Instance->HTR and LTR.


### Interrupt priority
Interrupt priority is critical and must be taken into account when setting up the system. The order should go:
-1 Reset, NMI
0 Physical fault handlers (busfault etc...) and the hardfault handler. These should be populated with a generateBreakAll() function to immediately disable all motors if they are reached.
1 (optional) Timer1/8 BRK interrupt - This will already have turned off the PWM through hardware link.
1 (optional) ADC watchdog interrupt (should only ever be reached in an out of range event, and should be populated with a handleError(&mtr[n], ERROR_ADC_OUT_OF_RANGE_IA) or generateBreakAll().
2 Timer1/8 Update - This calls the fast and hyperloops and is the main control loop.
3 Slowloop timer
...
5-onwards USB, UART, DMAs, SPI, systick... in any order of your choosing.




### Hall
Hall sensors only commutation is supported, but only in forward mode (swap a pair of motor phase wires if it runs backwards) but full support will be gradually deprecated since sensorless works much better. 
Hall startup option relies on a different mechanism and work in either direction by preloading the observer with flux linkages derived during Tracking.
Timer XOR hall input is not currently used, instead the fastloop just samples and counts PWM cycles since the last hall state change, which is fed into a filter. This reduces accuracy, but improves reliability, noise rejection and portability.


### Encoder
TLE5012B crudely supported in SSC (SPI) mode. The SPI reads are called synchronously from the hyperLoop and therefore PWM frequency is limited to about 20kHz to allow time for the data transfer.
ABI not currently supported. 


### PWM Input
Timer2,3 or 4 can be set up in reset mode, with prescaler sunch that one count = 1us (e.g. for F303 at 72MHz we would use a prescaler of 71. We use a 65535 period, with trigger/reset mapped to TI1FP1, and channel one capturing on rising edge, direct mode, channel 2 capturing on falling edge indirect mode - remapped to TI2FP2. This gives two CC register values, CC1 timing the period of the pulses, and CC2 timing the on time.  
Interrupt: update interrupt used. We expect a value very close to 20000us from an RC sender (50Hz). Check CC1 is 20000+/-~10000. If outside this bounds, set input capture flag low. CC2 is the pulse time.


### Over Current Comparators


#### On F303 based hardware 
Comparators set up to trigger Tim1 break2 state in the event of overcurrent event, which should turn off all outputs to high impedance on F303 hardware. 
0.5mOhm shunts (2x1mohm) or 1mOhm (2x2mOhm original hardware) at 100amps gives 50mV, with a pullup to 100mV. Vrefint is 1.23V, so 1/4Vref used for comparator-ve - 310mV. This triggers the comparator at 400A nominal (a LOT of current, but the intended FETs are rated for that for 100us, which is ~2 PWM periods... If alternate FETs used, should check this, or just hope for the best, or modify the shunt resistors to have higher value.  
Timer1 should have a BRK filter set to avoid switching noise  


#### On most other hardware 
On F4, L4 and other targets, the expectation is that there is a fault input on the PB12 pin for timer1,which is setup to capture and stop PWM generation on this input. Timer8 is mapped to PA6, annoyingly removing an ADC channel.
Hardware without overcurrent comparators is OK, the overcurrent and over voltage is also taken care of in the fastloop, or by the analog watchdog if set up.


## Watchdog timer
Not currently implemented. Lack of it has not presented any issue so far. ToDo...
If implementing specifically for your hardware,set it running in the hardwareInit() function. The watchdog timer should be kicked by the fast control loop after the VIcheck is completed to ensure a response to overvoltage and current events is possible.
Period of ~1ms 
On overflow, generate a break state on the motor and reset MCU - control loop no longer running, motor could be stopped, freewheeling, generally making a mess of currents and generating high voltages.


## General workflow


### Fast Control loop
Fast control loop must:  
Retrieve current and Vbus values from ADC 
Carry out the conversion to floating point SI units
Check for over limit events if not handled in hardware
Retrieve phase voltage values if not modulating
Process the state machine (Idle, running, tracking, error... and sensor mode)  
Manipulate currents and voltages to Vab (Clarke transform) and Vdq(Park transform)(FOC)  
Calculate current position and speed (get Hallsensor values, sensorless observer)  
Run FOC PI controller
Run Field weakening and circle limiter
Run SVPWM (include inverse clark/park)
Update inverter PWM values based on phase and voltage  


### Hyper Control loop
Hyper control loop has more optional things, which can include:
Process HFI
Read encoder
Interpolate the angle
IF logging: write currents and voltage to logging buffer, increment/wrap logging buffer index 




### Slow control loop
Execute every 10ms. Originally this ran on the RCPWM input, but not it runs independently. 
The slowloop can be run at any speed from about 20Hz to 1kHz on any timer with interrupt priority lower than the fastloop timer and ADC interrupt (if used)
State machine processing (ToDo)


# Control Loops


## Fast Control loop
All the FOC gets done in the fastLoop. This is the only critical part of the MESC, the rest can actually be removed providing the parameters get set and MESC is initialised correctly .
If you remove the slowloop, you can write directly to mtr[n]->FOC.Idq_req.q, set the mtr[n]->MotorState to MOTOR_STATE_RUN and enable the inverter.


### The geometric transforms
The forward transforms take place in the ADCConversion(_motor) function. This converts the read currents firstly from 3 phase to two phase (Clarke) and then rotates them to the estimated/read from encoder rotor reference frame (Park).


Geometric transforms are the Clark and Park, which take the forward form are used to transform currents:


Clarke:
\\[\begin{bmatrix}I_\alpha \cr I_\beta \cr I_\gamma \end{bmatrix} = 2/3 \begin{bmatrix} 1 & -0.5 & -0.5\cr0 & \sqrt{3}/2 & -\sqrt{3}/2 \cr 0.5 & 0.5 & 0.5 \end{bmatrix} \begin{bmatrix}I_u \cr I_v \cr I_w \end{bmatrix}  \\]
where MESC selects for the lower of the PWM values at high modulation using the substitution \\( I_u + I_v + I_w = 0\\) .


Park (rotation matrix around \\( \gamma\\)):
\\[ \begin{bmatrix}I_d \cr I_q \cr I_0 \end{bmatrix} = \begin{bmatrix}cos\theta & sin\theta & 0 \cr -sin\theta & cos\theta & 0 \cr 0 & 0 & 1\end{bmatrix} \begin{bmatrix}I_\alpha \cr I_\beta \cr I_\gamma \end{bmatrix}\\]
Where \\( \theta\\) is the electrical angle of the rotor; the mechanical angle divided by pole pairs and where the bottom and right rows of the matrix are ignored (we assume \\( \gamma\\) is zero).


The backward, or inverse, form of the transform is used on the voltages to create a 2 phase stator reference and then a 3 phase stator reference voltage from the 2 phase rotor reference. These are performed in the function writePWM(_motor) which is run in the fast AND hyperloop to enable voltage injection for HFI.


Inverse Park:
\\[ \begin{bmatrix}V_\alpha \cr V_\beta \cr V_\gamma \end{bmatrix} = \begin{bmatrix}cos\theta & -sin\theta & 0 \cr sin\theta & cos\theta & 0 \cr 0 & 0 & 1\end{bmatrix} \begin{bmatrix}V_d \cr V_q \cr V_0 \end{bmatrix}\\]
Inverse Clarke:
\\[ \begin{bmatrix}V_u \cr V_v \cr V_w \end{bmatrix}=  \begin{bmatrix} 1 & 0 & 1\cr -0.5 & \sqrt{3}/2 & 1\cr 1-.5 & -\sqrt{3}/2 & 1 \end{bmatrix} \begin{bmatrix}V_\alpha \cr V_\beta \cr V_\gamma \end{bmatrix}\\]
MESC uses the full form of the inverse clark where many other implementations skip it and use a SVPWM routine. MESC does this to enable a variety of clamping and over modulation methods, and because it is easier to understand, with no unexplained leaps of faith.
The end result is identical.


\\( sin\theta\\) and \\( cos\theta\\) are calculated from a lookup table 320 elements (=256x1.25) long optionally with interpolation. With interpolation, the maximum error from this is very small; less than the ADC or PWM resolution.


### The Sensorless Observer
The MESC sensorless observer is also known as the MXLEMMING observer and is now the default on the VESC project.
#### What MESC Does
The sensorless observer is very simple. The implementation is unique to MESC and was developed without recourse to appnotes or papers. It is probably not unique in industry, but so far I have not seen it in any other open source or commercial source project.
It works (as most successful observers do) on the basis of flux integration, that is the assumption that for a spinning magnet passing a coil, the voltage is given by:
\\[V = turns x \frac{d\phi}{dt} \\] 
and we observe from watching the motor on a scope that the voltages are sinusoidal.


Therefore, in general, if we ignore the number of turns and make \\(\theta = \omega t\\):
\\[ V = \phi\omega sin(\omega t)\\]
\\[\int V dt = turns \ast \phi +C -> -\phi\cos\theta +C\\] 
We do not need to care for turns, and C varies only dependent on where we start the integration for a sin wave.
The key recognition is that \\( \phi \\) is a constant dependent on the magnets, and therefore the max and min of the resulting integral are symetric and constant.
Since the voltage is sinusoidal, the flux integral will thus also be sinusoidal, with a phase shift of 90 degrees.
(remember to insert pics of sin and integral...)
Further, the addition of noise on the incoming voltage signal is effectively filtered out by this integral since 
\\[ \int cos(n\theta) dt = \frac{cos(n\theta)}{n} (+C) \\] 
and so noise and higher harmonics are greatly reduced.


Within MESC, we choose to carry out this integral in alpha beta frame, so we first remove the effects of resistance and inductance, and then integrate the resulting voltage as:
\\[ V_\alpha = V_{BEMF\alpha} + Ri_\alpha + \frac{Ldi_\alpha}{dt}\\]
\\[ V_\beta = V_{BEMF\beta} + Ri_\beta + \frac{Ldi_\beta}{dt}\\]
where \\(V_\alpha \\) and \\(V_\beta \\) are the electrical voltage output by the inverter and \\(i\alpha\\) and \\(i\beta\\) is the clarke transformed current measured by the ADC.
Thusly, we generate two estimated back EMF voltages which we can integrate to get two flux linkages with a 90 degree phase shift.


\\[ \phi_\alpha = \int V_{BEMF\alpha} dt \\]
\\[ \phi_\beta = \int V_{BEMF\beta} dt \\]


We have to deal with teh +C term in the integral, and also with integration drift which would result in arctangent not working. MESC simply clamps the flux integral at hard limits which can either be fixed or calculated in realtime by the flux linkage observer. 
Since they are shifted by 90 degrees and already filtered by integration, we need only find the arctangent of the two to calculate an estimated angle.
\\[ \theta = arctan(\frac{\phi_\beta}{\phi_\alpha}) + \pi\\]


#### Alternatives MESC chose not to do
Alternative to treating the inductance as a piecewise integral, the Lia term can be lumped. This would remove the need to store previous state information to calculate \\( \frac{di}{dt}\\) and is the method commonly used in literature. 
However, this allows the Lia term to get large compared to the back EMF, and the bounding/elimination of integrational drift is done while the inductance term is still within the BEMF, with probable impact on the result (note the result becomes clearly unstable when \\(Lia>phi\\))
Noteably the Ortega observer as used originally in VESC contains this construction, and relied on a non linear (quadratic) elimination of integration drift and associated instability at high current.


Alternative to the clamping of the flux integrals at their max possible limits a proportional (or PI or non linear) correction factor could be introduced based on the magnitude of the current alpha and beta fluxes. This is similar to the Ortega observer. MESC includes a version of this that can be compiled in with a #define USE_NONLINEAR_OBSERVER_CENTERING but it is advised you do not use this; for experiment only.


Alternatively to the arctangent we could construct a true observer:
\\[ \theta est_{n+1} = \theta est_n + d\theta + k_p*(\theta calc-\theta est)\\]
Where:
\\[ d\theta_{n+1} = d\theta_n + k_i*((\theta est_n + d\theta)-\theta calc_{n+1})\\]
(here we calculate \\(\theta calc \\) through arctangent as above and forward predict/correct our prediction each cycle)


Or: 
\\[ \theta est_{n+1} = \theta est_n + d\theta + k_p*\phi_d\\]
where
\\[ d\theta_{n+1} = d\theta_n + k_i*\phi_d\\] 
(here we use the d axis flux linkage, which we derive from a rotation of the alpha beta flux linkage as an estimate of the error to be corrected)


MESC chooses not to use a true observer, since there is no obvious measurable advantage, there are gains to be "tuned" which can result in instability with a true observer and there are additional calculation steps.
Noteable users of true observers include ST Micro's FOC library which uses a Leunburger observer and Alex Evers' UNIMOC which uses a Kalman filter.
Using a true observer of any kind does not deal with the three most fundamental problems facing sensorless observers: 
* Initially estimating parameters R and L, 
* Changing resistance with temperature and 
* Changing inductance with saturation at high current.


#### The MESC Salient Observer
MESC contains an observer for salient motors, which accounts for the differing d and q inductances. This is not usually required, is not well tested and will not work for outrunner motors since they saturate so heavily.
It relies on the assumption that the salience travels with the dq frame, and can be transformed into the alpha beta frame, then:


\\[ \frac{dLi}{dt} = \frac{Ldi}{dt} + \frac{idL}{dt} \\]


And therefore the above estimates for VBEMF can be modified to account for this changing salience in alpha beta frame.


### The FOC PI
The FOC PI is very simple. It is found in MESCfoc.c in the function MESCFOC(_motor).


It takes the current measurements in dq frame (they were previously collected from the current sensors and transformed by the Clarke and Park transform), calculates an error relative to the PI reference input (mtr[n]->FOC.Idq_req) and applies a change to the output voltage through a proportional and integral gain.
The gains are in units of "Volts/Amp" for the proportional and per second for the integral


The PI is a series PI, that is, the integral gain acts on the output of the proportional gain so:


\\[I_{err} \begin{bmatrix}d \cr q \end{bmatrix} = (I_{req} \begin{bmatrix}I_d \cr I_q \end{bmatrix} - I \begin{bmatrix}d \cr q \end{bmatrix} \times I_{pgain}\\]
and:
\\[I_{int-err} \begin{bmatrix}d \cr q \end{bmatrix} = I_{int-err} \begin{bmatrix}d \cr q \end{bmatrix} + I_{err} \begin{bmatrix}d \cr q \end{bmatrix} \times I_{igain} \times pwmperiod\\]
The output is then calculated as:
\\[\begin{bmatrix}V_d \cr V_q \end{bmatrix} = [I_{int-err} \begin{bmatrix}d \cr q \end{bmatrix} + I_{err} \begin{bmatrix}d \cr q \end{bmatrix}\\]
That's it. Nothing complex about the PI controller.


#### Gains
The trickier thing is how to set the gains, since it is quite possible to create gains that are orders of magnitude wrong, and wrong relative to each other. The target should be that there is response within a few PWM cycles; a few hundred us at most.


The gains can be calculated by setting a desired bandwidth ( mtr[n]->FOC.Current_bandwidth). The proportional gain is simply bandwidth*inductance and the integral gain is the ratio of inductance to resistance.


This is best examined in unit terms; bandwidth is \\(\frac{radians}{second}\\) inductance is \\(\frac{volts\times seconds}{amps}\\) giving pgain in units \\(\frac{volts}{amp}\\).


Likewise, if resistance is \\(\frac{volts}{amps}\\) and inductance is \\(\frac{volts \times seconds}{amps}\\) then Igain works out as \\(Igain = \frac{R}{L} = \frac{1}{seconds}\\).


Making the gains such means that the control loop is critically damped; the fastest reponse possible for a given bandwidth without overshoot.


### The Field Weakening
MESC runs two different field weakening methods, V1 is a dumb ramp of -Id between a starting duty and max duty. V2 is a closed loop field weakening system, where Id is added in response to reaching the duty threshold.
Both methods are run within the MESCfoc() function.
Both methods account for the total current allowed by reducing the Iq request in the slowloop as 
\\[ I_{qmax} = \sqrt{I_{max}^2-I_{FW}^2}\\]
Therefore, if you set lots of field weakening, the total motor current is conserved and you will not burn the coils (any more than you would otherwise, and you may still burn the core with greater iron losses)
#### The dumb ramp
Not much to say... it just ramps up the d-axis current with increasing duty. This is not always stable, if the ramp is too aggressive, it can cause reduction in duty which then causes field weakening to ramp down the next cycle and... oscillation.


#### The closed loop
A much smarter system implemented as a closed loop PI controller (proportional gain set to zero). As long as the bandwidth on the control is much lower than the current loop, it will be stable. That's not to say all motors and controllers are stable under field weakening.
If duty is greater/equal to than max duty:
\\[ I_{FW} = I_{FW} + 0.01 \times I_{FW-max}\\] 
else:
\\[ I_{FW} = I_{FW} - 0.01 \times I_{FW-max}\\]


Experiments with: 
\\[I_{FW} = K_p\times(duty-duty_{max}) + I_{FWint} \\] 
with:
\\[ I_{FWint} = I_{FWint} + K_i\times(duty-duty_{max})\\]
showed no improvement to stability or performance, and additional complication with gain tuning. It may be ressurected at a later date.


### The Circle Limiter
The circle limiter is a not so understood but absolutely critical part of the FOC controller. It's purpose is to ensure that the voltages sent to the inverter do not exceed what the inverter can actually produce, while keeping the PI controller happy. 


It must limit both the overall signal AND limit the PI integral, and do so in the 2D dq frame.


First, we must calculate the maximum length of the voltage vector, this is given by equating the hypotenuse of a 120 degree triangle to the bus voltage, then Vmagmax is the length of one of the shorter sides.


[insert picture of SV triangle]
\\[ V_{magmax} = \frac{1}{\sqrt{3}}V_{bus}*MAXMODULATION\\] 
Max modulation is a limit made to ensure there is always some PWM low time to ensure the bootstrap capacitors can recharge. If using isolated supplied, or overmodulation, this may not be required and you can set 1.0 or even 1.1 (beyond that it becomes quite unstable).


Typically, MAX_MODULATION is set to 0.95.


The identity:
\\[\sqrt(V_d^2+V_q^2)<=V_{magmax}\\]
should always be true.


To achieve this, MESC has two options:
#### Simple linear circle limiter
if: 
\\[\sqrt(V_d^2+V_q^2)>V_{magmax}\\] 
We can simply divide Vd and Vq by this value, and return them linearly to within the range of the circle.


Of course, we also need to ensure that the integral is not winding up in the background, and so we apply the rule:
if:
\\[|int_{Vderr}|>|V_d|\\]
\\[int_{Vderr}=V_d\\]
And similarly for Vq.
This is the default mode for ST Motor control library; it is the most obvious and easiest to implement solution.


#### Vd preferencing circle limiter


There may be reasons to prefer Vd to Vq. One such reason is that when we apply the field weakening, we need to ensure there is sufficient voltage available to generate the d axis current. Since the field weakening current is typically set lower than the torque current, a linear implementation of the circle limiter will result in reduced d axis current with increasing throttle.


To fix this, we give preference to the d-axis, but only up to a point, since with a fixed amount of available voltage, we find that forever favouring Vd results in a rapid drop in Vq as we pass Vd=Vq.


This results in collapse of the q axis current and its ability to control itself.


Presently MESC uses 60degrees as the threshold, i.e. 
\\[V_{dmax} = V_{magmax}*sin(60)\\] 
and therefore implicitly (not calculated!)
\\[V_{qmax} = V_{magmax}*cos(60)\\]
Hard coded as 0.866 (and implicitly 0.5). 


We firstly reduce the d axis voltage as:
\\[ if(V_d > V_{magmax}*0.866) \\]
\\[V_d = V_{magmax}*0.866 \\]
and then apply the circle limiting rule:
\\[V_{qmax} = \sqrt{V_{magmax}^2-V_d^2}\\]
And apply the q limit:
\\[if(V_q > V_{qmax}) \\]
\\[V_q = V_{qmax}\\]
And thusly we have an overall voltage magnitude that does not exceed the max voltage circle.


We apply the same rule to the integral terms as in the linear case to avoid windup.


ST make a variant of this for their motor control library as a selectable option. It does not include the limitation on the Vd proportion, but is otherwise very similar.


#### Things MESC does NOT do
There are blog posts from TI showing improved limiters that avoid current overshoot. They rely on reducing the integral term by the proportional term, with no regard for the ki. This is able to cause large jumps in the integral term with a gain of kp*input noise.


VESC implements the circle limiter this way, and it works, but it is possible to induce fluctations at high modulation. I think TI might have missed out the ki term from the integral limit feedback in their blog.


It is possible to implement this as a PID controller which provides a softer cutoff as the circle limit is approached. So far, I have not seen any reason to attempt this.


It is also possible to set the limits hard by precalculating the required Vd and Vq given the inductance and expected max velocity. This was the original MESC concept and worked well, but always resulted in not quite reaching max modulation, and therefore a few % drop in speed.


There are various non linear possibilities where exceeding the circle limit might have a quadratic rollback, or the the Vd and Vq might be bounded by a higher order flat bottomed polynomial. One day, if the present implementation is found limiting, MESC might adopt some other technique... until then, the method is conservative truncation.


### Tracking
Tracking is simple and relies on phase voltage meaurement.


The phase voltages are measured when PWM is disabled, and these measurements are Clarke transformed to get ab frame voltages, then Park transformed to get dq frame voltages.


The ab frame voltages are used in the flux observer to track the angle and the dq frame voltages are used to preload the integral component of the current controller PI so that when it restarts PWM there is no discontinuity.


The PWM is re-disabled every time the tracking loop is run to ensure that entering tracking from any state is safe (except in the case of heavy field weakenning when the slow loop does not move to tracking until the field weakenning current has dropped). Nothing more to it.


### The Hall start


Hall startup can be set using:


define USE_HALL_START


define HALL_VOLTAGE_THRESHOLD x.y


efine HALL_IIRN 0.0x


The hall start preloads the observer flux integrals with fluxes as monitored during the TRACKING state using a low pass infinite impulse response filter. In the tracking state, there is no current flow and therefore the estimate of the fluxes is unaffected by resistance and inductance; they are, broadly correct.


When the observer is subsequently called, the fluxes are biased towards the average value during that hall state, and thus the angle is strongly biased towards this. The flux integration continues to be carried out during this state, and so when the hall start is switched (at a defined voltage level) the sensorless observer is already running and accurately tracking.


The only parameter to be set is the IIR filter constant, which is by default 0.02 or 50PWM cycles, roughly 60eHz at 20kHz PWM. A faster motor will need a larger value (fewer PWM cycles). Less accurately set resistance or less accurate current measurements might require that you reduce the HALL_IIRN value.


## Porting to other MCUs
Intention is that minimal things have to be done to port:
1)         a) 3 phase (complimentary if required)PWM out required. Timer1 typically used, Timer8 can be used but main loop currently would need manually changing.
        b) set up CUBE with minimum 4x ADC readings (3x phase current and voltage compulsory, phase voltage sensors optional but strongly recommended and input for throttle if required)
        c) The ADC readings must be triggered at top center of the PWM, which can either be triggered through CCR4 or the update event.
        d) The readings can either be made by the Injected or regular conversion manager. Recommended that the injected is used for JDR1 = Iu, JDR2 = Iv, JDR3 = Iw, JDR4 = Vbus.
        e) The ADC can be set up to include an out of range watchdog which can make a rapid overcurrent protection. This is hardware specific, set AWD interrupt and thresholds in CUBE, and populate the ADC interrupt handler with handleError(ERROR_ADC_OUT_OF_RANGE_IA)
        f) The mapping between the ADC readings and the FOC inputs are made in the getRawADC() function, which is implemented per project. Replicate appropriately, using example from F405 project.
2) populate the timer1 update interrupt with         MESC_PWM_IRQ_handler() and __HAL_TIM_CLEAR_IT(&htim1,TIM_IT_UPDATE);
3) populate timer 3 or 4 interrupt with MESC_Slow_IRQ_handler(&htimx) and         __HAL_TIM_CLEAR_IT(&htimx,TIM_IT_UPDATE);
4) Run MESCInit() in the main
5) Create your own hardware conf file which includes the methods and settings and defaults for your hardware. 
6) Map your throttle and thermistor values in the getRawADC() function in your MESC_hw_setup.c file (replicate this from a similar chip is easiest)
7) Either hard code your motor parameters in your hardware header file, or implement the autodetection every time, or implement a variant of the flash reader/writer (this is the hard bit for new MCUs)
8) Resolve the includes - your project must point to ../MESC_Common/Inc in Properties ->C/C++ General->Paths and symbols
9) Populate the fault handlers with handleError(ERROR_HARDFAULT) and other appropriate fault names