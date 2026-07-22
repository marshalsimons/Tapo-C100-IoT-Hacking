# Tapo-C100-IoT-Hacking
Application of my previous IoT Hacking Research. This repository includes a detailed walkthrough of the methods and steps taken to exploit an IoT or embedded system.

## Background
In the last decade, Internet of Things (IoT) devices have become the majority of devices connected in each home. As a result, attackers have shifted their focus to IoT devices as targets because they lack the sophisticated defenses that modern operating systems provide. Additionally, because these devices are cheap, they often lack fundamental security by design before end users acquire them. Here, I will introduce offensive methodologies for exploiting IoT and embedded systems, and provide hands-on practice against the Tapo C100 camera.

## Methodology
Like all offensive engagements, there is no singular path to a target. Several avenues may need to be explored before finding the one that gains access to a system. Here, I will cover several methodologies for exploiting IoT and embedded systems, including Reconnaissance, Resource Development, Initial Access, Tools, and Software. 

### Target Acquisition and Reconnaissance
With a plethora of IoT devices on the market, choosing a target can be daunting. However, we can assume that an attacker would want their newly discovered vulnerable device to affect a large user base. To emulate the idea, we can simply grab the most popular IoT camera on Amazon. At the time of this research, the Tapo C100 Wi-Fi camera was the first item on Amazon available for approximately $25.

<div align="center">
<img width="300" height="400" alt="20260718_201700" src="https://github.com/user-attachments/assets/e410fa36-0af9-4b38-9ca9-a65acb438f54" />
</div>

Once a target is acquired, we can gather open-source information prior to tearing it apart. Most importantly, we can find a detailed breakdown of the board's topography and the manual. Both will contain critical information used to choose an attack vector and can be found easily through an internet search.

The FCCID website is a federal government-controlled site that contains information about essentially any device that broadcasts wireless signals. Here, we can find a breakdown of the board itself to identify chipsets and ports that could serve as exploitable vectors. The site also contains information about the wireless systems if a chosen attack vector. For this research, we will be exclusively focused on flash memory chips and UART debug ports.

https://fccid.io/TE7C100

Most devices will also include a detailed user’s manual directly from the vendor, which will be beneficial for identifying the connection layout for our flash memory chip. Mileage will vary when using these documents, so we will also walk through identifying ports and connections manually.

https://www.tp-link.com/us/support/download/tapo-c100/

### Firmware Attack Vectors
There are several attack vectors we can choose to exploit the C100. Our first target will be to acquire the device's firmware to identify vulnerabilities. We can get this file in a few ways, including downloading it from the vendor, sniffing the Flash memory signal, or connecting to the Flash chip directly.

Downloading the firmware from the vendor is the easiest method, but not all vendors will make it available. In our case, the C100 downloads its firmware updates directly from the vendor and does not release them online. If the firmware is not available from the vendor, we may be able to get it from someone who has dumped and uploaded it, but there is no guarantee the file will match our own. Alternativly, we can attempt to catch the frimware in transit as its pushed to the device by setting up a man-in-the-middle. This method is not yet included in this research and is a learning task for another day.

The second option is to sniff the signal from the flash chip when the device powers on. In embedded systems, the microcontroller is too small to hold the entire firmware. Instead, it uses a high-speed flash memory chip to store and read the data. When the device powers on, we can monitor the transaction between the flash chip and the MCU using a logic analyzer and dump it into our Saleae software. Doing so allows us to observe signal fluctuations in transmissions and reconstruct the firmware bit by bit. However, this method will be prone to error due to inconsistencies and bad connections. It is also rather tedious to do manually; however, there are readily available scripts to aid in the process.

Our last method will be to dump the firmware directly from the flash chip using the same programmers that the developers use to load and test it. This method will be more reliable and efficient than sniffing the traffic, but it may not always be an option.

### Shell Access
After we have found our exploitation path and are ready to gain shell access, we will create a connection to the device's debug UART ports. UART is the most common connection on IoT devices for debugging, and cheap devices are unlikely to have it disabled. We can leverage this feature if we can obtain the password for a privileged user. The easiest and most common way to gain access to a privileged user is to find the default password for the root user, as it is unlikely to have been changed or unique. They are also occasionally available online and in several repositories. Otherwise, we can extract the password hash from the firmware dump's/etc/passwd file and attempt to crack it.

### Software Exploitation
With the firmware dump, we can analyze the device's binaries to identify a software vulnerability to exploit. IoT devices are riddled with CVE’s that can be easy targets when identified. Fuzzers can look for unstable code, or automated tools can detect vulnerabilities.

I have used BugProve in the past to identify CVEs in a TP-Link router, but they have since moved to a submit-for-review platform. They do allow free users a limited number of scans per month.

https://bugprove.com/

Wairz is a newer tool using AI to identify vulnerabilities in firmware. At the time of this writing, the creator, Andrew Bellini, has taken it down for maintenance.

https://github.com/digitalandrew/wairz/blob/main/docs/index.md

## Tools
A series of inexpensive tools can be purchased from Amazon that provide multiple attack paths against an IoT device. Not all of these are required and can be downsized to a single vector. However, not all devices are the same and may require a unique approach.

<div align="center">
<img width="400" height="300" alt="20260718_201554" src="https://github.com/user-attachments/assets/8fcc748f-734e-4349-9a55-529e4e793e90" />
</div>

### Multimeter
A multimeter can be used to identify ports on an IoT device. We will use one to identify the UART TX/RX/GRND ports.

### Logic Analyzer
Logic Analyzers are optional tools that can sniff traffic signals generated by the flash memory chip.

### Flash Chip Programmer
Flash Chip Programmers will be among the most used tools in an IoT hacking toolbox, allowing direct connection to the flash chip. We will use ours to dump the firmware for analysis.

### UART Converter
The UART Converter allows us to connect to the board's debug terminal.

### Solder
A Solder may or may not be required depending on the UART ports. Some devices will still have developer-placed plugs, while others may have them removed and require new connections.

### Cables and Connectors
A wide range of cables and connectors is readily available. When connecting to the flash chip, you can use alligator clips, clothespin-style clips, or solder a connection. Make sure adjacent cables do not touch, or you risk shorting the device. 

## Software
There are several software options available to aid in IoT hacking. Here, we will use the most popular, open-sourced software available online. We will run everything on Ubuntu except Saleae, which will run on Windows. When running Saleae, you want the sampling rate to be at least 2x higher than the signal rate, which will require more resources than my Ubuntu VM has allocated.

### Saleae
Saleae is an open-source software for reading the signal captured by our logic analyzer. Our use case ends after examining the signal, but there is an optional attack vector that allows you to obtain the device's firmware solely by sniffing flash memory traffic. However, this method can be tedious and prone to error.

### Binwalk
Binwalk is a tool for enumerating a .bin file with several options for more in-depth analysis. We will use it to decompress the firmware and output it into several files for analysis. Binwalk also requires a few dependencies, which vary depending on the commands you use. If they are not included in your download, you will be prompted to download the package.

### Flashrom
Flashrom allows us to connect to the flash reader and dump the firmware. The connection may introduce errors, so it's recommended to grab several copies of the firmware and compare the file hash values to ensure we find a stable version.

### Screen
Screen is optional; any software that allows us to open a serial port connection to our UART converter will work.

## Walkthrough
Our first step is to crack the device open, access the PCB, and remove the camera to create some space.


<div align="center">
<img width="300" height="400" alt="20260718_202501" src="https://github.com/user-attachments/assets/92700d1c-fe8a-4c30-9bf9-aedd28158a20" />
</div>

Next, we will want to identify the flash chip and the MCU. The font can be hard to read, so you can either use a microscope or a phone camera.


<div align="center">
<img width="417" height="379" alt="image" src="https://github.com/user-attachments/assets/d8e6f693-c101-41cf-b9ca-70b76ff7227d" />
</div>

With the chip model in hand, we can search for the manual to identify the orientation and order of the connections. We will want to connect the device's CS, MISO, MOSI, and clock pins to synchronize our logic analyzer. The CLK is the “heartbeat” of the device and gives us our baud rate. The MISO (Master Input Slave Output) and MOSI (Master Output Slave Input) are the two primary communication lines on the chip. Lastly, the CS (Chip Select) picks which enslaved ship to communicate with.


<div align="center">
<img width="608" height="325" alt="image" src="https://github.com/user-attachments/assets/15a984e3-4c20-4565-a887-1e953f69fae8" />
</div>

Connect the logic analyzer and start Saleae to initiate the capture.


<div align="center">
<img width="300" height="400" alt="20260718_204505" src="https://github.com/user-attachments/assets/f0df01f0-0742-4ab9-9ef3-94e3f1ca308e" />
</div>

For settings, set the sampling rate to at least 2x the transmission speed, or as high as the device supports. For testing, I ran mine at 24 MHz. Rename the channels in the software to match the physical connections on the logic analyzer and apply the SPI analyzer with the corresponding channels. Lastly, start the capture and power on the device. The entire conversation will be relatively quick and complete in ~20 seconds, depending on the chip's transmission speed. If we zoom in, we can see the four channels and their corresponding values. For our purposes, we will stop here and move on to using a flash programmer. You can also attempt to reconstruct the firmware from the dump, which will be a learning task for me another day.


<div align="center">
<img width="1667" height="704" alt="Logic 2 Capture" src="https://github.com/user-attachments/assets/54eabce2-27f1-4939-9967-a94229167556" />
</div>

We will connect the flash programmer the same way we use the logic analyzer, with the addition of providing power to the chip with an extra pin.


<div align="center">
<img width="300" height="400" alt="20260719_173711" src="https://github.com/user-attachments/assets/381bf0e5-589c-427d-88d5-e5d2e500adf3" />
</div>

Once we are connected, we can power up flashrom and start dumping and analyzing the firmware.


<div align="center">
<img width="1411" height="530" alt="Screenshot 2026-07-19 141141" src="https://github.com/user-attachments/assets/39d3a8c4-5c71-44d5-8853-8069c0bc86a0" />
</div>

Now that we have the firmware, let's start looking around for some sensitive information. We will start with the SquashFS file system; devices sometimes contain the password and shadow files. We can use the data duplication tool to extract the files and decompress them. 

NOTE: I did not capture a screenshot showing the file being initially parsed with Binwalk. To get the output in the photo below, enter: **binwalk inputfile**


<div align="center">
<img width="1647" height="280" alt="binwalk1" src="https://github.com/user-attachments/assets/cf668079-cc17-453e-9826-f8e37d4517d7" />
</div>

Data duplication to extract only the squashfs files…


<div align="center">
<img width="939" height="122" alt="Screenshot 2026-07-19 142000" src="https://github.com/user-attachments/assets/5668be2c-1689-4d5d-9d59-48b947435b46" />
</div>

Decompressing the squshfiles…


<div align="center">
<img width="1643" height="309" alt="Screenshot 2026-07-19 142047" src="https://github.com/user-attachments/assets/2a84d8d1-5530-44e1-a743-4e06ddb95ab3" />
</div>

Standard Linux-based file hierarchy and searching for the shadow file...

<div align="center">
<img width="966" height="282" alt="Screenshot 2026-07-19 150545" src="https://github.com/user-attachments/assets/f7990949-c7b6-4b99-849f-c4f8974c3635" />
</div>

Unfortunately, the developers have kept the password and shadow files in another part of the firmware, which requires deeper analysis. Fortunately, another researcher had already played around with this device and had a handy command to search for the password recursively. Shoutout: Landon Crabtree.

<div align="center">
<img width="863" height="428" alt="Screenshot 2026-07-19 145651" src="https://github.com/user-attachments/assets/a78430ff-171a-40cd-a43f-562e18398c8b" />
</div>

With the root user password in hand, we can attempt to crack it. However, brute-forcing a hash can be difficult with certain algorithms, as the password may be too complex to crack within a reasonable time. So first, we can conduct research to see if it has been posted anywhere. Default credentials are relatively easy to find because they are seldom changed and are frequently reused. For our device, we are in luck, as it was already found to be root/slingenic. 

Now we can move on to getting a shell by connecting to the UART port. While not all boards will be the same, a UART can be easily identified by its 4 contact pads in a row. Some devices still have the pins attached, which will save some soldering. On other boards, the ports may be scattered and require manually checking of each pad until you find them.

<div align="center">
<img width="489" height="402" alt="image" src="https://github.com/user-attachments/assets/8b9f81ae-7e8a-4d85-bcd8-dfdb25263077" />
</div>

UART runs off 3 connections: a transmit (TX), receive (RX), and ground (GRND). Ground can be found by using the multimeter on any metal portion of the board and the pad to detect continuity. You will know you have the right one when the multimeter beeps.

<div align="center">
<img width="300" height="400" alt="20260719_152909" src="https://github.com/user-attachments/assets/16c1a54b-057e-4ae6-a836-0b1dd6fe9ae1" />
</div>

The transmit pin can be identified by finding the one with a ~3.3V reading that fluctuates during device power-up.

<div align="center">
<img width="300" height="400" alt="20260719_152302" src="https://github.com/user-attachments/assets/34463f0e-a0da-4006-82fb-c7d659716452" />
</div>

The receive pin can be harder to identify and will require some trial and error. However, we can make an educated guess that it is the contact pad between the ground and transmit pins. Next, we will solder cables to the contact pad and connect them to our UART converter. This was my first time soldering, and on my first attempt, I killed the board. I found that I had too much solder between the ground and the receive pin that likely caused it to short when I powered the device on. To prevent this from happening again, you can check the continuity between each port to ensure they are not connected in any way. You can also check continuity from the soldered port to your cable end to verify a good connection. 

<div align="center">
<img width="300" height="400" alt="1000015587" src="https://github.com/user-attachments/assets/7642e91b-a0a4-4236-a472-84d92e3e6a8f" />
</div>

Next, connect the soldered cables to your UART converter and spin up the UART shell using screen. You will also need to specify the connection's baud rate, which can sometimes be guessed by cycling through common rates manually (9600, 19200, 38400, 57600, 115200). You can also try to determine the rate with a multimeter or by researching the manual, but we were able to gain access with 115200.

<div align="center">
<img width="1296" height="752" alt="root_access" src="https://github.com/user-attachments/assets/fec8164d-7fde-409b-b76e-7cf628308e23" />
</div>

Immediately, you will see many errors flashing in the shell. After some time, there will be a 16-second pause, during which we can quickly check the running process. PID 143 /bin/main was consuming the most memory and was likely the cause of the traffic, so we can kill it and keep it out of our way for now.

<div align="center">
<img width="880" height="933" alt="ps" src="https://github.com/user-attachments/assets/5071f193-42a3-4d0d-a086-9dab9613e76d" />
</div>

With the log traffic out of the way, we can start to enumerate the device. We are not going to go in-depth on this device at this time; instead, we are only seeking basic information that could be useful for offensive operations. First, we will check who we are logged in as and parse the directory.

<div align="center">
<img width="793" height="182" alt="initial access and id" src="https://github.com/user-attachments/assets/f2020ef4-1ef3-4b60-a132-c20da9208c44" />
</div>

Grabbing patch information about the operating system and firmware is also useful in identifying published vulnerabilities. This device only has one CVE at the time of writing that allows DoS attacks by supplying crafted web requests: https://www.thehackerwire.com/vulnerability/CVE-2023-39610/.

<div align="center">
<img width="822" height="46" alt="os version" src="https://github.com/user-attachments/assets/931f4f4b-af97-4be0-9aa1-a55dac3a5e88" />
</div>

Even though we already found the contents of the passwd file in the firmware, we can take another look at the passwd and shadow files to see if there are any other users on the device. Unfortunately for us, all other users are disabled besides the root user. 

<div align="center">
<img width="666" height="161" alt="passwd" src="https://github.com/user-attachments/assets/ba52a528-fcd8-4693-9b11-12b622e8b9be" />
</div>

Next, we will check the binaries installed on the device. These are excellent targets to fuzz for vulnerabilities that could enable additional exploitation methods. Further analysis also found that the device runs a single monolithic process (/bin/main), and any vulnerable binaries could allow full access to the entire device's software. 

<div align="center">
<img width="919" height="993" alt="etc fiel" src="https://github.com/user-attachments/assets/dc8b8789-a153-4d91-bd4c-cff3c100e694" />
</div>

Lastly, we will check the private key pair the device uses to encrypt its data in transit. Unfortunately, since I fried my other device, I do not have another device to compare the keys against and determine whether they were unique. While they should be created uniquely on first power-up, some IoT devices have been found to use a single, shared key generated when the firmware was last loaded. You can verify this by checking the timestamp when the key was signed and comparing it to the firmware. If the timestamp matches the one when you first booted the device, it is highly likely to be unique. However, if it matches the firmware's date, it is likely that all similar devices share the same key pair and are vulnerable to decrypting TLS/SSL traffic, as well as the RTSPS protocol used for streaming camera data. Coincidentally, the C500 camera from the same vendor, Tapo, was found to have this vulnerability: https://nvd.nist.gov/vuln/detail/CVE-2025-1099.  

Disclaimer: Sharing your private key online, as I do below, is a very bad idea if you plan to use the device. I have mine contained within a cyber range with no external network access, and I will not use it for consumer purposes.

<div align="center">
<img width="664" height="774" alt="key pair" src="https://github.com/user-attachments/assets/1ae3106f-bc1c-4e28-a812-b88dbdc5417f" />
</div>

## Findings
IoT devices like the Tapo C100 lack many of the security practices found in more advanced systems. Default credentials, enabled debug ports, and unencrypted firmware allow attackers with basic offensive skills to gain unauthorized access to a root terminal. An attacker could easily deploy malware through these connections and return the device to the store, awaiting a victim to purchase it. The lack of security measures also means they can more easily serve as a point of ingress to adjacent machines with stronger security. Lastly, weak security designs, such as hard-coded keys and monolithic process design, introduce additional software vulnerabilities that can compromise the device's confidentiality, integrity, and availability for consumers. 

To remediate these vulnerabilities, developers should implement security-by-design principles and reduce the devices' attack surface and vulnerability score. While keeping the firmware unavailable online and sending it only directly to the device as it's pushed reduces its availability, it does not remove the possibility of acquisition by bad actors. Implementing encrypted firmware would serve as an additional barrier for attackers to overcome, but would also incur greater computational overhead. Disabling the UART ports would also prevent attackers from gaining access to the console, but this should be weighed carefully, as it would also prevent legitimate users from accessing their device if need be. Default credentials and unique key pairs are critical vulnerabilities with little to no drawbacks in mitigating, and they should be addressed when the device is still in production. 

## Future Plans
I plan to continue this work in an upcoming class on software exploitation, where I plan on focusing on the differences in efficiency and results between manual and AI-assisted firmware exploitation. This project was geared toward implementing something I had researched during an offensive security class and following through the steps myself. 

