# Ryzentosh (AMD Ryzen 5 1600X & MAXSUN RX 580 2048SP)

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
