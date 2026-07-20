# Tapo-C100-IoT-Hacking
Application of my previous IoT Hacking Research. This repository includes a detailed walkthrough of the methods and steps taken to exploit a IoT or embedded system.

## Background
In the last decade, Internet of Things (IoT) devices have become the majority of devices connected in a given home. As a result, attackers have shifted their focus to IoT devices as targets since they lack the sophisticated defensives modern operating systems ship with. Additionally, the fact that these devices are cheap also means they often lack security by design fundamentals prior to end users acquring them. Here, I will introduce offensive methodoliges that allow the exploitation of IoT and embedded systems, as well as the hands practice against the Tapo C100 camera. 

## Methodology
Since there are a plethora of IoT devices on the market, choosing a target can be a daunting task. However, if an attacker discovers an exploitation for a device, they would likely want to have large pool of victims to target. To emualte the idea, we can simply grab the most popular IoT camera on Amazon. At the time of this research is the the Tapo C100 Wi-Fi camera for approxiamtly $25. 

### Target Acquisition
Once a target is acquired, we can gather open source information prior to tearing it apart. Most importantly include a detailed breakdown of the board to choose an attack vector and the manual which will contain critial information. Both of these documents can be found easily enough through a internet search. 

The FCCID website is a federal goverment controlled site that contains information about essentially any device that broadcasts wireless signal. Here, we can find a breakdown of the board itself to identify chipsets and ports that can be exploitable vectors. The site also contains information about the wireless systems if a chosen attack vector. For this research, we will be exclusivly focused on flash memory chips and UART debuf ports.

https://fccid.io/TE7C100

Most devices will also include a detailed users manual directly from the vendor which will be benefitical for identifying the connection layout for our flash memory chip. Miliage will very using these documents, so we will also walk through identifiing ports and connections manually.

https://www.tp-link.com/us/support/download/tapo-c100/

### Tools

#### Multimeter
#### Logic Analyzer
#### Flash Chip Reader
#### UART Converter
#### Solder
#### Cabels and Connecters
### Software
#### Binwalk
#### Flashrom
#### Screen

## Attack Vectors
### Firmware
#### Downloading From Vendor
#### Flash Sniffing
#### Flash Reader

### Shell Access
#### UART

### Software Exploitation
#### BugProove
#### 
#### Manual Enumeration

## Walkthrough
## Findings
## Next Steps

