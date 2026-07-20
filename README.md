# Tapo-C100-IoT-Hacking
Application of my previous IoT Hacking Research. This repository includes a detailed walkthrough of the methods and steps taken to exploit an IoT or embedded system.

## Background
In the last decade, Internet of Things (IoT) devices have become the majority of devices connected in each home. As a result, attackers have shifted their focus to IoT devices as targets since they lack the sophisticated defensives modern operating systems ship with. Additionally, the fact that these devices are cheap also means they often lack security by design fundamentals prior to end users acquiring them. Here, I will introduce offensive methodologies that allow the exploitation of IoT and embedded systems, as well as the hands practice against the Tapo C100 camera. 

## Methodology
Since there are a plethora of IoT devices on the market, choosing a target can be a daunting task. However, if an attacker discovers an exploitation for a device, they will likely want to have large pool of victims to target. To emulate the idea, we can simply grab the most popular IoT camera on Amazon. At the time of this research is the Tapo C100 Wi-Fi camera for approximately $25. 

### Target Acquisition
Once a target is acquired, we can gather open-source information prior to tearing it apart. Most importantly include a detailed breakdown of the board to choose an attack vector and the manual which will contain critical information. Both of these documents can be found easily through an internet search. 

The FCCID website is a federal government controlled site that contains information about essentially any device that broadcasts wireless signals. Here, we can find a breakdown of the board itself to identify chipsets and ports that can be exploitable vectors. The site also contains information about the wireless systems if a chosen attack vector. For this research, we will be exclusively focused on flash memory chips and UART debug ports.

https://fccid.io/TE7C100

Most devices will also include a detailed user’s manual directly from the vendor which will be beneficial for identifying the connection layout for our flash memory chip. Mileage will be very using these documents, so we will also walk through identifying ports and connections manually.

https://www.tp-link.com/us/support/download/tapo-c100/

### Firmware Attack Vectors
There are several attack vectors we can choose to exploit the C100. Our first target will be to acquire the firmware of the device to identiy vulnerabilites. We can get this file a fwe different ways, inclduing downloading it from the vendor itself, sniffing the Flash memory signal, or connecting to the Flash chip directly. Downloading the firmware from the vendor is the easitst method, but not all vendors will make their available. In our case, the C100 downloads its firmware updates directly from the vendor and does not release it online. If the firmware is not avaialble from the vendor we may be able to get it from someone who has dumped and uplaodded it, but there is no guarantee the file will match our own.

The second option is to sniff the signal from the flash chip when the device powers on. With embedded systems, the microcontrller is to small to contain the entirity of the firmware. Intead, it uses high speed flash meemory chip to store and read the data. When the device powers on, we can monitor that transaction between the flash chip and MCU using a logic analzyer and dump it into our Saleae software. Doing so allows usto see the flucuations of the signal represneting the data in transmissions and reconstruct the firmware bit-by-bit. However, this method will be prone to error through inconsistenceis and bad connections. It was also be rather tedious to do manualy, however there are readily avaialble scripts to aid in the process. 

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

#### Multimeter
A multimeter can be used to identify ports on an IoT device. We will use one to identify the UART TX/RX/GRND ports.
https://www.amazon.com/gp/aw/d/B01ISAMUA6/?_encoding=UTF8&pd_rd_plhdr=t&aaxitk=fe86b462330dee4d40ad46f63a1334d1&hsa_cr_id=0&qid=1784573281&sr=1-1-9e67e56a-6f64-441f-a281-df67fc737124&i=aps&aref=f0RxgTwj4j&ref_=sbx__sbtcd_asin_0_title&pd_rd_w=S0W48&content-id=amzn1.sym.2fb72bc8-96ef-420d-b08f-c04b69f36507%3Aamzn1.sym.2fb72bc8-96ef-420d-b08f-c04b69f36507&pf_rd_p=2fb72bc8-96ef-420d-b08f-c04b69f36507&pf_rd_r=GVTZEYFVT3ZMBZQNNV9B&pd_rd_wg=re6DO&pd_rd_r=1147f512-5c3e-40a5-aa06-4f605508597d&th=1

#### Logic Analyzer
Logic Analyzers are an optional tool that can sniff the traffic signals generated by the flash memory chip. We will also use the Saleae software to capture and analyze the binary signals
https://www.amazon.com/LONELY-BINARY-Logic-Analyzer-Channels/dp/B0FKGN3JMZ/ref=sr_1_2_sspa?crid=D6FEMH7IRIGU&dib=eyJ2IjoiMSJ9.UTbf-HCPJ3wsUXwXdi8IdxCvqo7uG7Iv4MFz__QiaFwsuJF7ZrVS2xvnroRMdiHSTVL2anuR0sHQhTymJRW1plUJuBb6hBSnUNj5jFAyI3c.kITWY_BSn8BdXd6rLfJV24gD3y0dNMyJCg_ZGY86GCg&dib_tag=se&keywords=Logic+Analyze&qid=1784573298&s=hi&sprefix=%2Ctools%2C93&sr=1-2-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1

#### Flash Chip Programmer
Flash chip Programmers will be one of the most used tools in an IoT hacking toolbox that allows for direct connection to the flash chip. We will use ours to dump the firmware for analysis.
https://www.amazon.com/WWZMDiB-CH341A-Programmer-Removable-Organizer/dp/B0BYSRSZPC/ref=sr_1_2?dib=eyJ2IjoiMSJ9.K5HsZMlcGvmT3np1YXvwEeggoINv8_YHI0On05KSwQe4vUjBPDJPqNPqCuGxHz_ln5QhecHMvZtZ8l6-93MwU_meBD8ST3jYHReg-4Y9P3raGdQTXNyALk41tPWdu2_eCCUiQF3te-GC6CECoFqbT4vbPleremINYmlEg5E0Jr8ZV5-nSlwmQ5CgBLgwiZ7Uw1ueVkXIzkQQRg1MJ2WMou9Tlp7u52S7kFkSXwygAtnWwwDFLkhYfwWzjieT16r5fqKzHGOCFxnPFkrtqBa3Dn9Pq8WHOwbyHlvxlQuGlbI.7ELKAaDUAKgDk7FtmwosrKy7MxdSx7k2vJbS0SraU1U&dib_tag=se&keywords=Flash+Chip+Programmer&qid=1784573314&sr=8-2

#### UART Converter
UART Converter allows us to connect to the debug terminal on the board itself.
https://www.amazon.com/DSD-TECH-SH-U09C5-Converter-Support/dp/B07WX2DSVB/ref=sr_1_1_sspa?crid=4BP9JEU5Y33Z&dib=eyJ2IjoiMSJ9.5IphDe4no_Aq6JWc-6Az71Q8I1cXBAmdH1fJkjm5HdHfMr-WDloZPqTbtm9Iopw5nJevsI_x_Jm_SmUHvSJ1AWdPWi8en9N2rZUxoe-GJFmL8SPSE6LESNr_-FPE0R_qeyu0EXQN4W2U8tpYwnjJVrnIkRP5WaNiuTy2Dp8D9e-RxH9hpruhtM3IoWHfGoT-dq2C1x_x9XHKHzo36Cn3_v83476J4D6SljlPgnRe-8M.MMUM6FVMJrHuUNqBGvhA4_PWcKEaWxObddViiPuE4VY&dib_tag=se&keywords=%23%23%23%23+UART+Converter&qid=1784573333&sprefix=+uart+converter%2Caps%2C281&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1

#### Solder
A Solder may or may not be required depending on the UART ports. Some devices there will plugs still in place from the developers while others may have them removed and need connections soldered. 
https://www.amazon.com/Soldering-Interchangeable-Adjustable-Temperature-Enthusiast/dp/B087767KNW/ref=sr_1_2_sspa?crid=19905YQ6U3IR9&dib=eyJ2IjoiMSJ9.QUpD02drwVJlGcRsfXRvSDjFA3KYlwQjmlMFr78ZQcV1nYR4Xu7OUbISBbdip-dqPlgUaHpoD8E9kPsyYcTOowl_dhidy05Fk6ekz5VoOaYdEZEXpziUoZW8yolkDPeAcl-HJiCAybXXrOvCae854onWPrGeOfLj1C5SEWsBKLA8sRi7V-96vnhy6Jos2ArU52xWOT7aCUXLXHdmoXbhh-ogzVsH1KutYOEFJkIZ5PQMtA04fCXpenMGr5itaVnW7rs_b6Ni_VztS-pWvbD-fpxkEIEXpgnqZh3owuV5rak.8hGlPrNcGrisGDIAQxNTMq2D6VfhG_wV8eCdr6L9vSs&dib_tag=se&keywords=solder&qid=1784573345&sprefix=so%2Caps%2C394&sr=8-2-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1

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
## Findings
## Next Steps

