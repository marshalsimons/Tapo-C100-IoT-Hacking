# Tapo-C100-IoT-Hacking
Application of my previous IoT Hacking Research. This repository includes a detailed walkthrough of the methods and steps taken to exploit an IoT or embedded system.

## Background
In the last decade, Internet of Things (IoT) devices have become the majority of devices connected in each home. As a result, attackers have shifted their focus to IoT devices as targets since they lack the sophisticated defensives modern operating systems ship with. Additionally, the fact that these devices are cheap also means they often lack security by design fundamentals prior to end users acquiring them. Here, I will introduce offensive methodologies that allow the exploitation of IoT and embedded systems, as well as the hands practice against the Tapo C100 camera. 

## Methodology
Since there are a plethora of IoT devices on the market, choosing a target can be a daunting task. However, if an attacker discovers an exploitation for a device, they will likely want to have large pool of victims to target. To emulate the idea, we can simply grab the most popular IoT camera on Amazon. At the time of this research is the Tapo C100 Wi-Fi camera for approximately $25. 

<img width="300" height="400" alt="20260718_201700" src="https://github.com/user-attachments/assets/e410fa36-0af9-4b38-9ca9-a65acb438f54" />

### Target Acquisition
Once a target is acquired, we can gather open-source information prior to tearing it apart. Most importantly include a detailed breakdown of the board to choose an attack vector and the manual which will contain critical information. Both of these documents can be found easily through an internet search. 

The FCCID website is a federal government controlled site that contains information about essentially any device that broadcasts wireless signals. Here, we can find a breakdown of the board itself to identify chipsets and ports that can be exploitable vectors. The site also contains information about the wireless systems if a chosen attack vector. For this research, we will be exclusively focused on flash memory chips and UART debug ports.

https://fccid.io/TE7C100

Most devices will also include a detailed user’s manual directly from the vendor which will be beneficial for identifying the connection layout for our flash memory chip. Mileage will be very using these documents, so we will also walk through identifying ports and connections manually.

https://www.tp-link.com/us/support/download/tapo-c100/

### Firmware Attack Vectors
There are several attack vectors we can choose to exploit the C100. Our first target will be to acquire the firmware of the device to identiy vulnerabilites. We can get this file a fwe different ways, inclduing downloading it from the vendor itself, sniffing the Flash memory signal, or connecting to the Flash chip directly. Downloading the firmware from the vendor is the easitst method, but not all vendors will make their available. In our case, the C100 downloads its firmware updates directly from the vendor and does not release it online. If the firmware is not avaialble from the vendor we may be able to get it from someone who has dumped and uplaodded it, but there is no guarantee the file will match our own.

The second option is to sniff the signal from the flash chip when the device powers on. With embedded systems, the micrcontroller is to small to contain the entirity of the firmware. Intead, it uses high speed flash meemory chip to store and read the data. When the device powers on, we can monitor that transaction between the flash chip and MCU using a logic analzyer and dump it into our Saleae software. Doing so allows usto see the flucuations of the signal represneting the data in transmissions and reconstruct the firmware bit-by-bit. However, this method will be prone to error through inconsistenceis and bad connections. It was also be rather tedious to do manualy, however there are readily avaialble scripts to aid in the process. 

Our last method will be duming the firmware directly from the flash chip itself using the same programers the devlopers use to load and test it. This method will be more reliable and efficent than sniffing the traffic, but may not always be an option.

### Shell Access
After we have found our exploitation path and ready to gain shell access, we will create a conection to the devices debug UART ports. UART is the most common connection on IoT devices for debugging and cheap devices are unlikely to have them disabled. We can levarege this feature if we are able to gain the password for a privledged users. The easyist and most common way for these devices will be finding the default password for the root user as it unlukely to have changed or been unique. Otherwise we can find the password has from the /etc/passwd file in the firmware dump and attempt to crack it. 

### Software Exploitation
With the frimware dump we can analzye the binaries included on the device in an attempt to find a software vulnerability to exploit. IoT devices are riddled with CVE's that can be easy targets when identified. Fuzzers can look for unstable code or automated tools can detect vulnerabilites. 

I have used BugProove in the past to identify CVE's in a TP Link router but they have since moved to a submit-for-review platform. They do allow free users limited scans a month. 
https://bugprove.com/

Wairz is a newer tool using AI to identify vulnerabilites in frimware. At the time of this writing the creator Andrew Bellini has it taken down for maintenance. 
https://github.com/digitalandrew/wairz/blob/main/docs/index.md

### Tools
A series of inexpensive tools can be purchased from amazon that allow multiple pathways to attack a IoT device. Not all of these are required and can be downsized to a single vector. However, not all devices are the same and may require a unique approach. 

<img width="400" height="300" align="center" alt="20260718_201554" src="https://github.com/user-attachments/assets/8fcc748f-734e-4349-9a55-529e4e793e90" />

#### Multimeter
A multimeter can be used to identify ports on an IoT device. We will use one to identify the UART TX/RX/GRND ports.

#### Logic Analyzer
Logic Analyzers are an optional tool that can sniff the traffic signals generated by the flash memory chip. We will also use the Saleae software to capture and analyze the binary signals

#### Flash Chip Programmer
Flash chip Programmers will be one of the most used tools in an IoT hacking toolbox that allows for direct connection to the flash chip. We will use ours to dump the firmware for analysis.

#### UART Converter
UART Converter allows us to connect to the debug terminal on the board itself.

#### Solder
A Solder may or may not be required depending on the UART ports. Some devices there will plugs still in place from the developers while others may have them removed and need connections soldered. 

#### Cabels and Connecters
A wide range of cables and connecters are readily available. When connecting to the flash chip, you have the option of using alligator plugs, clothespin style clips or even soldering a connection.

### Software
There are several software options to choose from to aid in IoT hacking. Here we will use the most popular and open-source software available on the internet. Select Linux distro can run all the tools listed here. We will be running everything out of Ubuntu except for Saleae, which will be on Windows. When running Saleae you want the sampling rate to be at least 2x higher than the signal rate which will require more resources than my Ubuntu VM has allocated.  

#### Saleae
Saleae is an open-source software for reading the signal captured by our logic analyzer. Our use case will end after looking at the signal but there is an optional attack vector where you attain the device firmware solely from sniffing the flash memory traffic. However, this method is tedious and prone to error.  

#### Binwalk
Binwalk is a tool for enumerating a .bin file with several options for more in-depth analysis. We will use it to decompress the firmware and output it into several files for analysis. Binwalk also requires a few dependences varying based on the commands you use. If they are not included in your download, it will prompt you with the package to download. 

#### Flashrom
Flashrom allows us to connect to the flash reader and dump the firmware. There can be some induce errors from the connection, so its recommended grab several copies of the firmware and compare the file hash values to ensure we find a stable version.

#### Screen
Screen is optional as any software that allows us to open the serial port connection to our UART converter and port will work.

## Walkthrough
Our first step is to crack the device open to gain access to the PCB board and remove the camera to remove some space.

<img width="300" height="400" alt="20260718_202501" src="https://github.com/user-attachments/assets/92700d1c-fe8a-4c30-9bf9-aedd28158a20" />

Next, we will want to identify the flash chip and MCU. The font can be hard to read, so you can either use a microscope or a phone camera.

With the chip model in hand we can search for the manual to idetify orientation and order of the connections. We will want to have the device CM, MISO, MOSI, and clock to synchronize our logic analzer. The CLK is the "heartbeat" of the device for a single clockcycle, while the MISO (Master Intput Slave Output) and MOSI (Master Output Slave Input) show the back and forth coversation from the chip and MCU. The chip select picks which slave to communicate with.

<img width="300" height="400" alt="20260718_204505" src="https://github.com/user-attachments/assets/f0df01f0-0742-4ab9-9ef3-94e3f1ca308e" />

Match the connections to the logic analyzer and start Saleae to initate the capture. For settings, you want the sampling rate to be a minimum of 2x higher than the transmission speed or as high as the device suppots. For my testing I ran mine at 24mHz. Rename the channels in the software to match the physcal connections on the logic analzer and apply the SPI analzyer with the corresponding channels. Lastly, start the capture and power the device on. The entire conversation will be relativly quick and complete in ~20seconds dependant on the chip tranmission speed. If we zoom in we can see the four channels and their corresponding values. For our purposes, we will stop here and move onto using a flash programmer. You can attempt to reconstruct the firmware from the dump, however that will be a learning task for myself another day.

<img width="1667" height="704" alt="Logic 2 Capture" src="https://github.com/user-attachments/assets/54eabce2-27f1-4939-9967-a94229167556" />

We will connect the flashprogrammer the same way we used the logic analzyer, with the addition of providing power to the chip with an extra pin. Once we are connected we can powerup flashrom and start to dump and analyze the firware.

<img width="1411" height="530" alt="Screenshot 2026-07-19 141141" src="https://github.com/user-attachments/assets/39d3a8c4-5c71-44d5-8853-8069c0bc86a0" />

The Squshfs file system sometimes can contain vulnerable files including the password and shadow files. We can use the data duplication tool to extact the files and decompress them.


<img width="1647" height="280" alt="binwalk1" src="https://github.com/user-attachments/assets/cf668079-cc17-453e-9826-f8e37d4517d7" />

<img width="939" height="122" alt="Screenshot 2026-07-19 142000" src="https://github.com/user-attachments/assets/5668be2c-1689-4d5d-9d59-48b947435b46" />

<img width="1643" height="309" alt="Screenshot 2026-07-19 142047" src="https://github.com/user-attachments/assets/2a84d8d1-5530-44e1-a743-4e06ddb95ab3" />

<img width="966" height="282" alt="Screenshot 2026-07-19 150545" src="https://github.com/user-attachments/assets/f7990949-c7b6-4b99-849f-c4f8974c3635" />

## Findings

## Next Steps

