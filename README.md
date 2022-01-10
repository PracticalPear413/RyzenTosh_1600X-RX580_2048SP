# Ryzentosh (AMD Ryzen 5 1600X & MAXSUN RX 580 2048SP)

## My Specs
CPU: Ryzen 5 1600X.  
GPU: MAXSUN RX 580 2048SP  
RAM: 8GB 3000MHz  
Motherboard/Laptop Make and Model: Gigabyte B450M DS3H    
Audio Codec: Realtek  
Ethernet Card: Realtek 
Wifi/BT Card: I haven't figured out the wifi part yet  

## Guide

Maxsun RX 580 2048SP does support spoofing but the hardware acceleration still doesn't work for some reason.
So the only solution is to flash BIOS on your AMD GPU. I know it sounds scary but fear not.

Boot into Windows and download GPU-Z, you'll find the GPU information here. Switch to the Advanced tab and select Memory and check what Memory chip you have.
I had Samsung so I had to look for RX 570 BIOS that have support for Samsung chip.
I found AMD Radeon RX 570 supported Samsung and the other specs were almost the same.

I looked for the BIOS that had similar frequency (not exactly the same) and the cores and everything were similar.
Here's my GPU-Z
<a href="https://imgbb.com/"><img src="https://i.ibb.co/K7Pk0Z9/image.png" alt="image" border="0"></a><br />
<a href="https://ibb.co/1mknFGh"><img src="https://i.ibb.co/G5qJzQB/image.png" alt="image" border="0"></a><br /><br />
MY ORIGINAL BIOS : https://www.techpowerup.com/vgabios/219359/219359
<a href="https://ibb.co/yBQgKWk"><img src="https://i.ibb.co/G35CGcM/image.png" alt="image" border="0"></a><br /><br />

and here's the RX 570 BIOS I found
https://www.techpowerup.com/vgabios/210814/amd-rx570-4096-170424
<a href="https://ibb.co/sVRBdRG"><img src="https://i.ibb.co/RDb5kbm/image.png" alt="image" border="0"></a><br /><br />

As you can see, the GPU clock is different but only by 44 MHz, the Device ID is obviously different and so is the power consumption but that won't really matter much.

So I used AMDVBFLASH utility to load the ROM and flash it, it flashed without any errors and took just 10 seconds. After a restart, it finally showed up as RX 570 and I could use MacOS with hardware acceleration.

Just copy the EFI folder in your OPENCORE USB and directly boot from it. I flashed MacOS catalina first but you can upgrade to Monterey after booting too.

## Unsuccessful Spoofing

I followed the guide here: https://dortania.github.io/Getting-Started-With-ACPI/Universal/spoof.html
The dortania guide mentions that you can get the ACPI path of the GPU using
```
$ cat /sys/bus/pci/devices/0000:01:00.0/firmware_node/path
```
but no such folder as firmware_node existed on my ZorinOS installation. I also tried other distros but no luck.
Booting into Windows, I found the path to be _SB.PCI0.GPP8 but this is incorrect as well, since it's not the full path. This will spoof incorrect path and you'll get an empty GPU show up in Hackintool alongisde your original GPU.
So to circumvent this, we need to create our SSDT-GPU-SPOOF.dsl file like this:

```
// Based off of WhateverGreen's sample.dsl
// https://github.com/acidanthera/WhateverGreen/blob/master/Manual/Sample.dsl
DefinitionBlock ("", "SSDT", 2, "DRTNIA", "AMDGPU", 0x00001000)
{
    External (_SB_.PCI0, DeviceObj)
    External (_SB_.PCI0.GPP8, DeviceObj)


    Scope (\_SB_.PCI0.GPP8)
    {
        if (_OSI ("Darwin"))
        {
            Device (GFX0) {
            Name (_ADR, Zero)
            Method (_DSM, 4, NotSerialized)  // _DSM: Device-Specific Method
            {
                Local0 = Package (0x04)
                {
                    // Where we shove our FakeID
                    "device-id",
                    Buffer (0x04)
                    {
                        0xDF, 0x67, 0x00, 0x00
                    },
                    // Changing the name of the GPU reported, mainly cosmetic
                    "model",
                    Buffer ()
                    {
                        "Radeon RX 570"
                    }
                }
                DTGP (Arg0, Arg1, Arg2, Arg3, RefOf (Local0))
                Return (Local0)
            }}
        }
    }
    Scope (\_SB.PCI0)
    {                   
        Method (DTGP, 5, NotSerialized)
        {
            If (LEqual (Arg0, ToUUID ("a0b5b7c6-1318-441c-b0c9-fe695eaf949b")))
            {
                If (LEqual (Arg1, One))
                {
                    If (LEqual (Arg2, Zero))
                    {
                        Store (Buffer (One)
                            {
                                 0x03
                            }, Arg4)
                        Return (One)
                    }

                    If (LEqual (Arg2, One))
                    {
                        Return (One)
                    }
                }
            }

            Store (Buffer (One)
                {
                     0x00
                }, Arg4)
            Return (Zero)
        }
      
    }

}
```
Notice that I'm wrapping the Method inside 

```
Device (GFX0) {
            Name (_ADR, Zero)
            ....
            ....
            
}
```
This makes sure the correct ACPI path to the GPU is selected. After updating the config.plist, I rebooted and it worked!
I was finally able to go from `Display 7MB` to `Radeon RX 580` but the problem was hardware acceleration still didn't work.
The spoof was perfect, it also improved the performance by a lot but still, hardware acceleration was missing for some reason.

I read that it's because of new Whatevergreen v1.5.5 which requires no-gfx-spoof in Device Properties in config.plist but that only made the OS not boot.
I tried multiple versions of WhateverGreen but it still didn't work, so I decided to use the flashing method instead.

If someone can figure this out, it'd be great but my methods didn't work.
