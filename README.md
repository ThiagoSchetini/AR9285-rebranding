# Dual Band AR5BHB92 wireless card rebranding for thinkpad

The idea here is to use a dual band Atheros AR5BHB92 AR9280 wireless card, and make it bypass thinkpads whitelist (warning: tested only on my T430)

Good News!
There is an Atheros that passes on Thinkpad t430 whitelist! I think that it works too on x230, w530 ...
`168C:002B:17AA:30A1`

How To?
We gonna modify the EEPROM of our AR5BHB92 to become this card. We gonna change the ProductID, SubProductID and SubVendorID to another one.


# How to buy this card exactly?

I've bought from this link, i recomend to buy exactly the same one:
`https://www.aliexpress.com/item/AR5BHB92-AR9280-Dual-Band-2-4G-5GHz-802-11a-b-g-n-300Mbps-WiFi-Wireless-Network/32792736484.html?spm=a2g0s.13010208.99999999.262.jRvcby`

Link not working anymore?
Please, try searching for this term on Ali Express, look for the one that has the waelab brand on it:
`AR5BHB92 AR9280 Dual-Band waelab`

You need the Ubuntu OS to make this!
You gonna need a Laptop with no wifi restriction, Ubuntu OS (any version), and of course, with the Atheros wireless card pluged in (and only this please)


# The easiest way ever!

I prepared this easy way only for AR5BHB92 of the Ali Express link, okay?

Considering the files on this repository, the iwleeprom is compiled, patched and ready to use!

Inside the iwleeprom folder, using iwleeprom tool:  
`sudo ./iwleeprom -i ../AR5BHB92-eprom/patched.eeprom`  

You're gonna see some messages of the write process. Some [.....X...] where X are the mods
... and some success write message on the final

That's all folks!

Your card is patched and ready! Put it back in the Thinkpad, and boot it. You don need the FAKE-PCIID...kext, you need only the toleda kext for atheros. Visit my t430 hackintosh repository:  
`https://github.com/ThiagoSchetini/macosx-thinkpad-t430`  

Note that your card still works on another OS like Linux and probably on Windows, because, it's an Atheros yet! 


# Another AR928x? Try the hard way, it's how i did it!

First we need to identify the wireless card
Then, in terminal:  
`lspci`  
This will give us the pci slot used by the cards, for me: 03:00.0  
Then we take a look at:  
`ls /sys/bus/pci/devices/`  
And we look for the same pci slot, for me: 03:00.0  
And then we use:  
`udevadm info /sys/bus/pci/devices/0000:03:00.0`  
Result:
```
P: /devices/pci0000:00/0000:00:1c.1/0000:03:00.0
E: DEVPATH=/devices/pci0000:00/0000:00:1c.1/0000:03:00.0
E: DRIVER=ath9k
E: ID_MODEL_FROM_DATABASE=AR9285 Wireless Network Adapter (PCI-Express)
E: ID_PCI_CLASS_FROM_DATABASE=Network controller
E: ID_PCI_SUBCLASS_FROM_DATABASE=Network controller
E: ID_VENDOR_FROM_DATABASE=Qualcomm Atheros
E: MODALIAS=pci:v0000168Cd0000002Bsv000017AAsd000030A1bc02sc80i00
E: PCI_CLASS=28000
E: PCI_ID=168C:002A
E: PCI_SLOT_NAME=0000:03:00.0
E: PCI_SUBSYS_ID=144F:7156
E: SUBSYSTEM=pci
```
This command give us enough informations about the card. We are interested in the PCI_ID and PCI_SUBSYS_ID. We'll modify both of them.  

```
We save those informations somewhere.  
So finally we have:  
- PCI_ID 168C:002A will become 168C:002B  
- PCI_SUBSYS_ID 144F:7156 will become 17AA:30A1 

TIP: If your card have different ID's, don't worry. Use the same logic and you gonan have the same result!

Now we will dump the AR9285 EEPROM into a file, modify that file(to make it look like whitelist Atheros), and copy it back to the wireless card.  
We need a few tools.  
`sudo apt-get install build-essential ghex`  
ghex is a hexadecimal editor.  
Then, we download the tool source code that allows us to read and write EEPROMs, iwleeprom:  
```
wget https://storage.googleapis.com/google-code-archive-source/v2/code.google.com/iwleeprom/source-archive.zip
unzip source-archive.zip
cd iwleeprom/branches/atheros
```
Then we need to modify three pieces of code inside ath9kio.c, around line 795:  
```
	if (dev->ops->eeprom_read16(dev, 128, &data) && (376 == data)) {
		short_eeprom_base = 128;
		short_eeprom_size = 376;
		goto ssize_ok;
	}
	if (dev->ops->eeprom_read16(dev, 512, &data) && (3256 == data)) {
		short_eeprom_base = 512;
		short_eeprom_size = 3256;
		goto ssize_ok;
	}
	if (dev->ops->eeprom_read16(dev, 256, &data) && (727 == data)) {
		short_eeprom_base = 256;
		short_eeprom_size = 727;
		goto ssize_ok;
	}
```
change into:  
```
	if (dev->ops->eeprom_read16(dev, 128, &data) && (376 == data)) {
		short_eeprom_base = 0;
		short_eeprom_size = 512;
		goto ssize_ok;
	}
	if (dev->ops->eeprom_read16(dev, 512, &data) && (3256 == data)) {
		short_eeprom_base =  0;
		short_eeprom_size = 512;
		goto ssize_ok;
	}
	if (dev->ops->eeprom_read16(dev, 256, &data) && (727 == data)) {
		short_eeprom_base = 0;
		short_eeprom_size = 512;
		goto ssize_ok;
	}
```
save, and compile:  
`make`  

Now make sure this is the AR9285 card that we are actually using.
We'll dump the AR9285 EEPROM into a file:  
`sudo ./iwleeprom -o ./AR9285-original.eeprom`  
Result:  
```
Supported devices detected:
  [1] 0000:03:00.0 [RW] AR9285 Wireless Adapter (PCI-E) (168c:002b, 1a3b:1089)
Select device [1-1] (or 0 to quit): 1
Using device 0000:03:00.0 [RW] AR9285 Wireless Adapter (PCI-E)
IO driver: ath9k
HW: AR9285 (PCI-E) rev 0002
RF: integrated
Checking NVM size...
ath9k short eeprom base: 0  size: 512
Saving dump with byte order: LITTLE ENDIAN
0000 [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx]
0f80 [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx]

EEPROM has been dumped to './AR9285-original.eeprom'
```

The file is owned by the root user, but you can change it back to a normal user owner 
`sudo chown username ./AR9285-original.eeprom`

We keep the original intact, and we'll work on a copy:  
`cp AR9285-original.eeprom AR9285-patched.eeprom`  

Now it's time to open the eeprom dump file with ghex, find and replace vendor/device/subsystem IDs:  
`ghex AR9285-patched.eeprom`  

Now something IMPORTANT, the eeprom dump file is byte-flipped, which means, if we want to look for "168C", we will look for "8C16".  

- PCI_ID 

168C:002A look for 8C 16 2A 00 replace by 8C 16 2B 00 (only the B changed on this case)

- PCI_SUBSYS_ID 

144F look for 4F 14 replace by A1 30

7156 look for 56 71 replace by AA 17 

WARNING: sometimes there are 2 OCCURENCES of each to find and replace by.

And now we write back the modified eeprom dump file into the wireless card, using iwleeprom tool:  
`sudo ./iwleeprom -i AR9285-patched.eeprom`  

Now our AR9285 card is patched and ready. We put it back in the Thinkpad T430, and boot it. You should not see any warning from the BIOS.  

That's all!

Your card is patched and ready! Put it back in the Thinkpad, and boot it. You don need the FAKE-PCIID...kext, you need only the toleda kext for atheros. Visit my t430 hackintosh repository:  
`https://github.com/ThiagoSchetini/macosx-thinkpad-t430`  

Note that your card still works on another OS like Linux and probably on Windows, because, it's an Atheros yet! 
