# Emulation of programming environment for single-board computer Raspberry Pi at monitoring the recurrent laryngeal nerve
>Mykola Dyvak,1, Volodymyr Tymets1
 Faculty of  Computer Information Technology, Ternopil National Economic University, UKRAINE, Ternopil, Chehova 8,E-mail:  volodymyrtymets@gmail.com
 
 
 __Abstract__ -  *In this paper the emulations of module for identification the recurrent laryngeal nerve by electrophysiological method via single board computer are given.*
	__Keywords__– *neck surgery, recurrent laryngeal nerve, rasbery pi, single-board computer.*
## INTRODUCTION
Nowadays, the numbers of operation for organs of neck constantly increases. The greatest threat, during this operations are injury of the recurrent laryngeal nerve (RLN). Therefore, to reduce the risk of using complex software engineering tools. The basic principle of operation of such assets is as follows. Stimulation of tissue surgical wounds or continuous alternating electric current. Fixation and software to process the results stimulation of to identify informative features the type of tissues.  
On the First Word Congress of Neural Monitoring in Thyroid and Parathyroid Surgery, which took place in Krakow in September 2015 [1], the variety of technical and software tools for identification the RLN positioning in a surgical wound are present. Functioning principle of these tools is based on the electrophysiological method of stimulation the tissues in surgical wound by a direct or alternate current with next registration and processing the results of stimulation with purpose of identification the informative characteristics of tissues kind. The new methods tools and mathematical models for the RLN identification are firstly considered in papers [2-5].
## TASK STATEMENT
In paper [8] it is described a scheme of application for new method and hardware for development the RLN monitoring system. This approach based of stimulating the tissues in signal wound by an alternating current. Then, result of simulation is recorded by sound sensor implemented in an endotracheal tube above vocal cords. The scheme of the method is shown in Fig.1
![](https://image.ibb.co/e5puo5/2017_04_19_20_45_12.png)
> 1 is respiratory tube, 2 is larynx, 3 is sound sensor, 4 are vocal cords, 5 is probe, 6 is surgical wound, 7 is alternator, 8 is amplifier, 9 is audio input and ou put card in computer

Fig. 1 Method of RLN identification among tissues in surgical wound.

In paper [6] is describing scheme of execution RLN method.
 In respirator tube 1 that inserted in larynx 2, the sound sensor 3 implemented and positioned above vocal cords 4. By means of probe 5 is connected to an alternator 7 with current strength from 0.5 to 2 mA and fixed frequency which provides small conductivity of electric signal by muscular tissues high conductivity of electrical signal a laryngeal nerve muscles witch control the tension of vocal cords.      
An a passing through a respiratory tube creates voice vibrations witch amplitude and spectrum are changing due to modulation the vibration of vocal cords in accordance witch frequency of simulation current. This vibration are registered by sound sensor 3 into electric signal and amplifying by amplifier 8 and sending to standard audio on sound card of computer 9 where this signal is processing.
This approach requires, a personal computer for processing the digital signal from the amplifier and generating an alternator. The disadvantage of this method, that he cannot be implemented as a single device. Therefore suggested to replace personal computer and alternator for single-board computers. The scheme of this approach is shown in Fig.2.
![](https://image.ibb.co/ipuoFk/2017_04_19_20_52_05.png)
>1 is respiratory tube, 2 is larynx, 3 is sound sensor, 4 are vocal cords, 5 is probe, 6 is surgical wound, 7 single-board computer, 8 is amplifier,

Fig. 2 Method of RLN identification among tissues in surgical wound, by using single-board computer.

Single-board computer, like alternator, should generate a signal for tissue irritation. After, received signal from amplifier also is processing by single-board computer. This allows remove cumbersome and redundant computer. Also this approach allowing to implement a method of RLN in a single device.

## MATHEMATICAL MODEL RLN

During the study, it was found that received information signal has random nature. This can be represented random set of nonrandom numerical characteristics of permanent or changing over time.
As is known, the autocorrelation function (ACF) periodic signal is also periodic function with the same period. So, she can be used to research the frequency of the output information signal.
ACF represents the degree of connection (correlation) between the signal u(t) and time shifted his copy of u(t-τ), where τ – time offset value signal.
To reduce the influence of noise components of the information signal, for his energy spectrum, finds ACF (1) for each selection signal [8-10]:

After building ACF for each segment of the information signal, use a Fourier transform (1). Then we obtain energy spectrum of information signal [7]:
## EMULATED ECOSYSTEM ENVIRONMENTS
To replace personal computer and alternator for single-board computers need choice something with sound card and output system to generate  a signal. The best choice for this is a Raspberry pi 3 model b. 
The Raspberry Pi is a series of small single-board computers developed in the United Kingdom by the Raspberry Pi Foundation to promote the teaching of basic computer science in schools and in developing countries[7]. The scheme of the Raspberry Pi is shown in Fig.3.
![](https://image.ibb.co/ikVZo5/2017_04_19_20_54_35.png)
Fig. 3 Shema of single-board computer Raspberry pi 3 model b.

Why Raspberry Pi? There are two giant upgrades in the Pi 3. The first is a next generation Quad Core Broadcom BCM2837 64-bit ARMv8 processor, making the processor speed increase from 900 MHz on the Pi 2 to up to 1.2GHz on the Pi 3.
The second giant upgrade (and this is the one we’re personally most excited about) is the addition of a BCM43143 WiFi chip BUILT-IN to your Raspberry Pi.  No more pesky WiFi adapters - this Pi is WiFi ready.  There’s also Bluetooth Low Energy (BLE) on board making the Pi an excellent IoT solution (BLE support is still in the works, software-wise)
This single board computer can work under control Linux or Windows 10 operating system.
The Raspberry pi 3 model bhas next list of characteristics: processor (Broadcom BCM2387 chipset, 1.2GHz Quad-Core ARM Cortex-A53,  802.11 b/g/n Wireless LAN; GPU (Dual Core VideoCore; memory (1GB LPDDR2).

## REALIZATION OF EMULATION OF PROGRAMMING ENVIRONMENT 
As said, in chapter above single-board computer Raspberry pi 3 model b can work under control Linux. References. Linux its generic name UNIX-like operating systems, based on the same core. It's totally free system based by Richard Stolman in 1983.
Via this operating system, can use Node.js and npm ecosystem. Node js is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient. Npm - node.js package ecosystem, is the largest ecosystem of open source libraries in the world.
To get accurate result ought, the  developed emulation module should work with reliable data. Since, not possible to work with real people used pre-recorded signals from patients. . The scheme of the developed module is shown in Fig.4.  
![](https://image.ibb.co/dj3b1Q/2017_04_19_20_55_34.png)

> 1 microfon, 2 sound recognize module, 3 wav file, 4 file decoder module, 5 sound encrypt module, 

Fig. 4 Schema of emulation of programming environment 

File decoder module allow convert sound wav file into 2D array, that first element of array it's time representation of signal, and second its amplitude of signal. Code of this module is given below.
```
const WavDecoder = require('wav-decoder');

const decode = fileName => {
  const filePath = path.resolve(__dirname, '../', 'assets', fileName);
  const readFile = filepath => {
    return new Promise((resolve, reject) => {
      fs.readFile(filepath, (err, buffer) => {
        return resolve(buffer);
      });
    });
  };
```

This decoder use wav-decoder package from npm ecosystem. Each package of npm it's part of code that wrote before to resolve some defined goal. Each package totally free, with open source code, and can be used for different goals.
Transforming sound signal happens in sound encrypt module via fast forward fourier transform method described above. 
```
decode('Crash-Cymbal-3.wav')
  .then(audioData => {
    const lowRange = ndarray(audioData.channelData[0]);
    const upRange = ndarray(audioData.channelData[1]);
    fft(1, lowRange, upRange);
    console.log('Love Signal Range ->', lowRange.data.slice(0, 3));
    console.log('Up Signal Range ->', upRange.data.slice(0, 3));
}).catch(err => console
```
Ndarray-fft package provide fast forward fourier transform method in node.js. This package can be used to do image, sound processing operations on big, higher dimensional typed arrays.
## RESULTS OF REALIZATION EMULATION OF PROGRAMMING ENVIRONMENT
The result of execution of this program is the same as in analog fft method in matlab.
Next results then were obtained:
* selected single-board computer to replace personal pc;
* developed test module, that work with real sound data of patients. This module can be runned in single-board computer Raspberry pi 3 model b. 
* in further wav file can be replaced for real sound recognition system. Sound signal will be inputted from microphone. Then his  will be passing for Rasbery py  sound card and prosesed by sound recognize module. After signal can processed by sound encrypt module.
##  CONCLUSION

Is offered replace personal computer and alternator for single-board computers. Shows the advantage of this approach. Elected and described the most appropriate for this aproach single-board computers Raspberry pi 3 model b.
Was designed and emulated temporary module processing audio signals. This module can be successfully launched on Raspberry pi 3 model b in futher.
## REFERENCES
[1] Abstract book of First World Congress of Neural Monitoring in Thyroid and Parathyroid Surgery, Krakow, Poland, September 20 l 5, 16 l p
 [2] VH. Riddell, "Thyroidectomy: prevention of bilateral recurrent nerve palsy," British Journal of Surgery, vo 57 no. 1, pp. 1-1 l, Jan. 1970 
[3] J. Galivan and C. Galivan, "Recurrent laryngeal nerve identification during thyroid and parathyroid surgery, Otolaryngology Head and Neck Surgery, vol. 112, no l 2, pp. 1286-1288, Dec. 1986 
[4] J.V. Basmajian, Muscles Alive Their Functions Revealed by Electromyography (5-th Ed.) Baltimore: Williams and Wilkins, 1985
[5] W E. Davis et al., "Reccurent laryngeal nerve localization using a microlaryngeal electrode", Otolaryngology Head and Neck Surgery, vol, 87, no. 3, pp. 330-333, May Jun. 1979 
 [6] M. Dyvak et al., Interval model for identification of laryngeal nerves," Przeglad Elektrotechni vol. 86 no. 1, pp. 139-140, 2010
[7].Bush, Steve (25 May 2011). "Dongle computer lets kids discover programming on a TV". Electronics Weekly. Retrieved 11 July 2011.
[8] M. Dyvak et al., “Spectral analysis the information signal in the task of identification the recurrent laryngeal nerve in thyroid surgery,” Przegląd Elektrotechniczny, vol. 89, no. 6, рр.275- 277, 2013. 
[9] M. Dyvak et al., “Interval model for identification of laryngeal nerves,” Przegląd Elektrotechniczny, vol. 86, no. 1, pp. 139-140, 2010. 
[10] M. Dyvak et al., “Algorithms of parallel calculations in task of tolerance ellipsoidal estimation of interval model parameters,” Bulletin of the Polish Academy of Sciences-Technical Sciences, Vol. 60, no. 1, pp.159-164, 2012. 
[11] V. P. Diakonov, MATLAB 6.5 SP1/7.0 and Simulink 5/6: Signal processing and filter design. Мoscow: Solon Press, 2005, 676 p.



