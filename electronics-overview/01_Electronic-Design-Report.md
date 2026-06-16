# **Design Report**
<div align="center">
<img width="1390" height="699" alt="image" src="https://github.com/user-attachments/assets/f7398aa1-2f95-4aa4-af10-29da4270c37a" />
</div>
<p align="center">(Figure 0) Top Layer Overview</p>


The carrier board is powered through a 5V USB-C cable that powers all the circuitry on this device. 
I chose this because I think USB-C feels better and is much easier to use compared to your typical 9V supply for pedals. 
It’s also just different, which is something I wanted to try. 
USBCC1 and USBCC2 are connected to pulldown resistors as they are required for a device to request power.
USBCD+ and USBCD- are connected but not used in this circuit. 
<div align="center">
<img width="1313" height="883" alt="image" src="https://github.com/user-attachments/assets/41828543-9e3a-4c6b-bd78-c0ba51f60011" />
</div>
<p align="center">(Figure 1) USB-C Connector </p>

Coming out of VBUS there is basic ESD protection and a decoupling capacitor which protects the rest of the board from static electricity and reduces unwanted AC noise. ESD protection is needed whenever circuitry is interfacing with the real world so that’s why I have one here and the input and as you will see later, one at the output
<div align="center">
<img width="830" height="758" alt="image" src="https://github.com/user-attachments/assets/d6559b73-cdea-40a6-a2c6-525e0657a692" />
</div>
<p align="center">(Figure 2) ESD Protection and Decoupling </p>

Next I’ll go into the various power branches. There is a VBUS branch, 5VA, 3V3, 3V3A which are used to power various circuits. I used the same LDO regulator to step down 5V to 3V3 volts. One is used for powering the (3V3) DAC and the other is used for powering the Digital supply on the ADC (3V3A). 3V3A is also used to power many other analog components such as the various OP-Amps that are used as buffers and amplifiers. I have a 5VA source which is just a filtered VBUS; the ferrite bead and capacitor form a low pass filter shunting high frequency noise to ground from the rail before it hits the ADC analog input. I also have some status LEDS and testpoints for quick testing and debugging. 

<div align="center">
<img width="865" height="894" alt="image" src="https://github.com/user-attachments/assets/3c645485-c20b-4b4c-8667-6f02b220e3b5" />
</div>
<p align="center">(Figure 3) Power Rails </p>

Originally this design only had one input for just a guitar, but since there were two audio lines (left and right), I decided I wanted to add in another input for a keyboard since it is my main instrument of choice. So here you can see the two inputs, one for guitar and one for keyboard. 
For each input signal, I have a standard  input jack wired to a pulldown resistor, ESD protection and a low pass filter and a coupling capacitor. Low pass filter here blocks everything above 7.234315MHz. This is for blocking any RF from the guitar cable since it has inductance. The coupling capacitor blocks any DC noise and allows the AC signal to pass through labeled GUITAR_FILTERED. The 0.1uF capacitor is used as a coupling capacitor blocking DC noise from our signal. 

<div align="center">
<img width="811" height="200" alt="image" src="https://github.com/user-attachments/assets/1ddac8d8-cb20-44ed-a9cf-9ea6b06b249f" />
</div>
<p align="center">(Figure 4.1) Input for Guitar </p>

<div align="center">
<img width="797" height="192" alt="image" src="https://github.com/user-attachments/assets/13e0a268-b5c4-4daa-a9eb-0c6801c5eff4" />
</div>
<p align="center">(Figure 4.2) Input for Keyboard </p>


Now the next step is where the process changes a little. A standard guitar signal is not very strong and needs gain, so in order to boost the input signal I used an opamp and configured it to have a gain of 6. For the keyboard we dont need to boost the signal since it is already at line level so I just send it through an OPAMP with a gain of one. There’s a great video on youtube that explains how we can properly use a single supply OPAMP for audio signals. The issue is that with single supply OPAMPS, our negative side voltage is tied to GND and the positive voltage is tied to a source (in this case 3V3) so the output can only swing from 0 < VOUT < 3V3. The issue that arises is that audio signals will swing above and below 0V so the entire bottom half will get clipped. 

<div align="center">
<img width="718" height="336" alt="image" src="https://github.com/user-attachments/assets/fca70e56-f8d8-4daa-a5b1-985e3571e068" />
</div>
<p align="center">(Figure 5.1) Audio Wave </p>

<div align="center">
<img width="1004" height="473" alt="image" src="https://github.com/user-attachments/assets/ee7ad369-3d3a-4fff-a6a0-cac2a8c700c9" />
</div>
<p align="center">(Figure 5.2) Audio Wave Clipped </p>

To avoid this issue, we can add a DC offset voltage that is half of the positive input voltage. This will allow the input signal to swing above and below a reference voltage without being clipped.

<div align="center">
<img width="1079" height="364" alt="image" src="https://github.com/user-attachments/assets/82011268-b73f-439a-ae5d-4e1e140c2e63" />
</div>
<p align="center">(Figure 6) DC Offset </p>

And so that’s what I did, I halved 3v3 using two 100k resistors and also had a big decoupling capacitor to protect against unwanted noise. This VREF is then passed through an opamp for a low output impedance (since w/o opamp,using the voltage divider gives an output impedance of 50k) and then “injected” to the AC signal through an injection resistor (1M ohms) into the positive input of the OPAMP. You can also see that on the negative input of the OPAMP we have this 10uF capacitor which acts as a high pass filter blocking DC signals at the input which prevents them from being amplified.  

<div align="center">
<img width="838" height="541" alt="image" src="https://github.com/user-attachments/assets/849d71fb-14a0-4da7-a006-baf4c34a16d8" />
</div>
<p align="center">(Figure 7) Bias Network for VREF </p>

<div align="center">
<img width="1178" height="687" alt="image" src="https://github.com/user-attachments/assets/8e16a696-e967-4ca3-b7f1-79f72cb974c4" />
</div>
<p align="center">(Figure 8) OPAMP Gain Stage with Injection resistor and Filters </p>

Now with this VREF_IN, our output voltage swings centered around 1.65V above to 3.3V and below to 0V. This first OPAMP is to boost the original signal and we do that with a gain of about 6. A typical guitar signal voltage can range from anywhere between 40mV and 500 mV on the higher end. My guitar is probably around 40mV to around 150mV so if we take our maximum possible input of 150mV and multiply by 6, we get 900mV and that is well within our range of +- 1.65V. If we do somehow manage to go over 275mV on the input signal then we will clip. 
Then I send it through a coupling capacitor to block DC noise and through another low pass filter (19.4KHz). This is because we typically sample audio at around 44.1 – 48kHz. The Nyquist frequency is the highest frequency one can sample without aliasing. Simply divide the frequency by 2, and you will get your value, for us it’s about 22-24kHz. So we need an anti aliasing filter that blocks anything above this. Our ADC has anti-aliasing built in so I just made a simple Low Pass filter using a resistor and a capacitor which is sufficient as it blocks everything above 19.4kHz. Then we pass it through a buffer for a low output impedance. This is for impedance bridging. When bridging a signal from one circuit to another, the output impedance to be as small as possible and the input impedance to be as large as possible. This is because, when you connect to circuits together, the output and input form a voltage divider and this can reduce our signal. So according to voltage divider math, we would want the output to be as small as possible and input to be as large.  We do this for both VIN_R and VIN_L.

<div align="center">
<img width="1336" height="476" alt="image" src="https://github.com/user-attachments/assets/58d06aa7-4c2d-40a2-abfb-1b8165bd22f8" />
</div>
<p align="center">(Figure 9) Impedance Bridging Theory </p>

<div align="center">
<img width="1035" height="631" alt="image" src="https://github.com/user-attachments/assets/afb5797a-049b-4e7e-be6b-160cf71dc0ad" />
</div>
<p align="center">(Figure 10) Filter + OPAMP Buffer </p>

Next I passed the signals through an ADC to convert it to a digital signal for the FPGA. Most of the ADC is just wired the way it is supposed to from the Datasheet. FMT is the audio format which we want to be I2S, and according to the datasheet tying it to ground configured for I2S 24 bit. MD01 and MD1 are two mode selects for master/slave mode and the clock ratio. Since we want it in slave mode and a 256*fs sample rate, we ground MD0, MD1 using a pulldown resistor. 
<img width="918" height="199" alt="image" src="https://github.com/user-attachments/assets/88920dff-aa34-45ba-84cc-51be06b64554" />
<p align="center">(Figure 11) Interface Modes </p>

SCKI, BCK, LRCK are all clock signals. SCKI is the system clock input, BCK is the bit clock, and the LRCK is the Left and Right clock. In slave mode, we want all these clocks to be coming from the FPGA so they are tied to FPGA pins and the FPGA outputs a signal to the ADC. DOUT is the digital signal sent to the FPGA via I2S. 

<div align="center">
<img width="1470" height="709" alt="image" src="https://github.com/user-attachments/assets/a6b2393d-087e-4fdb-a76b-c6c9db5607f8" />
</div>
<p align="center">(Figure 12) ADC Chip </p>

Next onto the FPGA. Here is the main controller of this entire board, it is the CMOD A7 with an Artix 35-T core. Some basics, I have put some buttons for functionality that I will determine later and Status LEDS for each of the I2S lines. I want to visually see if they are working correctly by outputting a signal correctly. I also have testpoints for each of those signal lines. If we are empowering through a 5V external source and somehow by chance the USB on the CMOD A7, then it requires a schottky diode so I placed one just to be safe. There are more decoupling capacitors again for blocking AC noise and also being able to power the FPGA if it needs a short burst of supply. Next all our signal lines are being fed into some GPIO pin of the FPGA. BCK and LRCK both go to the ADC and DAC. DOUT is the output from ADC being fed as an input into the FPGA and DIN is an output of the FPGA being fed into the DAC as you will see below. 

<div align="center">
<img width="1573" height="743" alt="image" src="https://github.com/user-attachments/assets/01a4dd7b-15b8-4a5c-b740-ead5db8ddeac" />
</div>
<p align="center">(Figure 13) FPGA Pinout </p>

The DAC chip is set up pretty standard to the data sheet. Using the 3V3 line from the LDO regulator we power this chip. We have the I2S LRCK and BCK lines fed from the FPGA into the DAC as well as DIN. Then we have the left and right channel output with its circuitry needed from the Datasheet. 

<div align="center">
<img width="1627" height="592" alt="image" src="https://github.com/user-attachments/assets/20e4dc8a-c0da-4462-bfbe-5416b980b93f" />
</div>
<p align="center">(Figure 14) DAC Chip </p>

Finally we have our analog output circuitry which is just a reconstruction filter to smooth out the digital signals into a nice smooth analog signal. Pretty similar circuitry to our input OPAMP circuits. We have a coupling capacitor that blocks DC noise and then a RC filter which is the crude version of a reconstruction filter. It is a low pass filter that blocks anything above 338 kHz. Fed into a unity gain buffer opamp for low output impedance then into a 100 ohm resistor to isolate the OPAMP output from any inductance of the output cable. We also have a coupling capacitor and then ESD protection and a bleed resistor. We have another bias circuit to get another VREF point so that we don't have to use the same one from the input side. 
<img width="1634" height="510" alt="image" src="https://github.com/user-attachments/assets/741faa09-6a03-41d0-8180-aa69a4b34a7f" />
<p align="center">(Figure 15) Analog Output </p>

