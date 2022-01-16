# Intro

This details the function and operation of the firmware. Wondering how the impedance measures are made? Want to get magnitude AND an accurate phase out of the device? Want to try different electrode stimulation patterns? This is the place to be. This branch requires IAR Embedded Workbench to run, and there is another gcc-toolchain branch for those interested. 

这详细说明了固件的功能和操作。想知道阻抗测量是如何进行的？想要获得设备的幅度和准确的相位？想尝试不同的电极刺激模式？这是要去的地方。这个分支需要 IAR Embedded Workbench 才能运行，另外还有一个 gcc-toolchain 分支。


# Hardware 
This firmware is designed for the Spectra: 32 electrode EIT device, which is also capable of bio-impedance spectroscopy and time series impedance measurements. It measures 2" square and is based on the ADuCM350 precision analog impedance analyzer chip. 

此固件专为 Spectra: 32 electrode EIT 设备设计，该设备还能够进行生物阻抗谱和时间序列阻抗测量。它的尺寸为 2 平方英寸，基于 ADuCM350 精密模拟阻抗分析仪芯片。

<p align="center">
	<img src="images/PCB.jpeg" height="200">
</p>

# Running the installed firmware. 

The device comes pre-programmed with this firmware. Each time you reset the device a set of menu options is communicated via UART or Bluetooth. Once you select an option, press the return key and that option will start executing until you power cycle once more. 

Options Outlines: 
A) Time series - 
   This outputs the impedance magnitude at 40 frames per second at 50kHz(or whatever frequency is set in firmware). 

   How to take a measurement with bioimpedance spectroscopy: A+, V+, V- ,A-

### Example of Use 
   
<p align="center">
	<img src="images/timeseriesexample.png" height="300">
</p>

B) Bioimpedance Spectroscopy - 
	This outputs impedance magnitudes at the following frequencies, and then repeats the cycle. 
	frequencies = {200,500,800,1000,2000,5000,8000,10000,15000,20000,30000,40000,50000,60000,70000};

### Example of use: 

<p align="center">
	<img src="images/spectrums.png" height="400">
</p>

C) Electrical Impedance Tomography - This is configurable in firmware to 8, 16, 32 electrodes dependent on desired timing and resolution trade offs etc. 

The ordering of electrodes is created using pyEIT as follows: 
ex_mat = eit_scan_lines(ne=8, dist=3)  step = 3
ex_mat = eit_scan_lines(ne=16, dist=8) step = 8
ex_mat = eit_scan_lines(ne=32, dist=16) step = 16
In the file the ordering is:
A+, A-, V+, V-
A, B, M, N
where
    A : current driving electrode
    B : current sink
    M, N : boundary electrodes, where v_diff = v_n - v_m

We have chosen an opposition based electrode patten instead of the more commonly used adjacent scheme, as we came across these papers supporting improved contrast if opposition schemes were used: 

[1]	A. Adler, P. O. Gaggero, and Y. Maimaitijiang, “Adjacent stimulation and measurement patterns considered harmful,” Physiol. Meas., vol. 32, no. 7, pp. 731–744, 2011.

[2]	F. M. Yu, C. N. Huang, F. M. Hsu, and H. Y. Chung, “Pseudo electrodes driven patterns for Electrical Impedance Tomography,” Proc. SICE Annu. Conf., pp. 2890–2894, 2007.

It's possible other electrode arrangement schemes are better yet. 

### Example of use

<p align="center">
	<img src="images/picturegrid.png" height="500">
</p>

A note on which debugger to use: 
- I recommend using a Segger J-link debugger with SWD programming cable. If you are not using it for commercial use you can get the EDU version at a reasonable price(you don't even have to be part of a school). The configuration on the PCB also allows for the VCOM port to work which means you can also read serial through the same connection. The specific configuration can be seen in the .bat file contained in the EXE directory. 

Behold my thorax! This was taken using this firmware, the most difficult part of the process was placing the electrodes.  

<p align="center">
	<img src="images/LungscomparedtoCTScan.png" height="300">
</p>



## Experimenting with the firmware

The best way to get experimenting with the firmware is to start with the Analog Devices example code for the ADuCM350(the main precision microcontroller that Spectra is based on) - https://ez.analog.com/analog-microcontrollers/precision-microcontrollers/w/documents/2411/aducm350-faq-evaluation-kit-software-platform  

试用固件的最佳方法是从 ADI 公司的 ADuCM350 示例代码开始（Spectra 所基于的主要精密微控制器）


Once you've got the Analog Devices examples working as a tutorial on how to get the environment set up, you can come back to this repository and play with stim patterns, any frequency you designate, or control the order bioimpedance measurements quite easily for your specific application. As mentioned, there are two branches of the firmware repository, one is a yet to be completed GCC port(feel free to contribute to a full port of the IAR branch into the GCC branch but might need some function testing to get it verified working), and the IAR Embedded Workbench IDE version in the main branch(I used V6.5) on Windows 10 through VMWare on my Mac Laptop, and a Segger J-Link to connect to the SWD.  

一旦您将 Analog Devices 示例用作有关如何设置环境的教程，您就可以返回此存储库并使用您指定的任何频率的刺激模式，或为您的用户轻松控制顺序生物阻抗测量具体应用。如前所述，固件存储库有两个分支，一个是尚未完成的 GCC 端口（请随意将 IAR 分支的完整端口贡献到 GCC 分支，但可能需要一些功能测试以使其验证工作） ，以及主分支中的IAR Embedded Workbench IDE 版本（我使用 V6.5），以及用于连接到 SWD 的 Segger J-Link。

## How to program the firmware the first time! 
I got a few questions on this one, so I added a folder called programming helper which gives you some example codes which I use on a raspberry pi to successfully program both chips. If you are programming the firmware, please look through it and try it, but here is a synopsis: 
Install Jlink commander - https://www.segger.com/products/debug-probes/j-link/tools/j-link-commander/

Ensure you have the ADuCm350 drivers and cfg file on your system(get this from analog devices). This maps out the memory space on your chip. 

确保您的系统上有 ADuCm350 驱动程序和 cfg 文件（从模拟设备获取）。这会映射出芯片上的内存空间。
```
cd "C:\Program Files (x86)\SEGGER\JLink_V630e"
JLink.exe -device ADUCM350 -if SWD -speed 115.2 -jtagconf -1,-1 -autoconnect 1 -CommanderScript %~dp0\JLinkCommandFile.jlink
```

This JlinkCommandFile, which is contained in the programming helper folder, can also just be typed into your jlink command prompt if you wish to execute it line by line. 
You connect to the chip, reset, erase. Now the chip is not spewing forth data and is ready to be programmed, so load up the new file, reset to give it a fresh start and it's all over. I suggest just doing this the first time, so you see how you can use JLinkCommander. 

这个 JlinkCommandFile 包含在编程助手文件夹(programming_helper)中，如果您希望逐行执行它，也可以将其键入到您的 jlink 命令提示符中。
您连接到芯片，重置，擦除。现在芯片没有喷出数据并准备好进行编程，因此加载新文件，重置以重新开始，一切都结束了。我建议第一次这样做，以便您了解如何使用 JLinkCommander。

If you are having an issue, I also suggest toggling the power switch to the other position, and then trying to program it again. 
Note: You do need to get yourself a Segger J-Link EDU version to do this, and plug it into the SWD header closest to the USB connector! 

如果您遇到问题，我还建议将电源开关切换到另一个位置，然后再次尝试对其进行编程。
注意：您需要为自己准备一个 Segger J-Link EDU 版本编程器来执行此操作，并将其插入最靠近 USB 连接器的 SWD 接头！

## Common Issue

A few people have had a hardware reset flag come up when trying to program the firwmare. You can obtain this if you have the SWD cable plugged into your J-Link the wrong way around. Please check that the red line matches with the 1 side of the plug. 

一些人在尝试对固件进行编程时出现了硬件重置标志。如果您将 SWD 电缆以错误的方式插入 J-Link，您可以获得此信息。请检查排线的红线是否与插头的 1 侧匹配。

## License 

The Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License seems to fit best with this project. Check out the human readable summary here: 

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.

If you'd like to make a derivative of this project in a commercial setting, we'd love some money so that we can afford to spend time maintaining this project and making more projects like this one, and obviously it's a share alike license too. If this hybrid open source model works, it would enable open source projects to receive some funding, making the global commons stronger to benefit everyone. It seems better for everyone if we can develop interesting things in the open, but what is the sustainable model to do this? 
