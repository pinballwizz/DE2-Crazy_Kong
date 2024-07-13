-------------------------------------------------
Crazy kong (Falcon) FPGA - DAR - 2014
-------------------------------------------------
Educational use only
Do not redistribute synthetized file with roms
Do not redistribute roms whatever the form
Use at your own risk

-------------------------------------------------
Update 2014 June 07 : Note

make sure to use ckongpt2.zip roms 
(MAME Crazy kong part II (set 1) - Falcon)
-------------------------------------------------

--------------------------------------------------------------------
External use of ram had been chosen for various reason :

 - demonstrate time slotted ram access 
 - allow project for very small fpga
 - allow for uploadable program/graphic upon static architecture
 - allow for delivering binaries without any roms data   

Of course this choice is not the more simple nor the more convenient
for large fpga. The choice of sdram against sram would have made this
project more versatile but (a little bit) less simple. 

--------------------------------------------------------------------
The original arcade hardware PCB contains 10 memory regions

 cpu addressable space
 
 - program                  rom  24Kx8, cpu only access
 - working ram              ram   3Kx8, cpu only access
 - color/sprite-data        ram   1Kx8, cpu + (2 access / 8 pixels)
 - background buffer        ram   1Kx8, cpu + (1 access / 8 pixels)
 - big sprite buffer        ram  256x8  cpu + (1 access / 8 pixels)        

 non cpu addressable region   

 - background/sprite graphics      rom 8Kx16, (1 access / 8 pixels) 
 - big sprite graphics             rom 2Kx16, (1 access / 8 pixels)
 - background/sprite color palette rom 64x8 , (1 access / pixels)
 - big sprite color palette        rom 32x8 , (1 access / pixels)
 - sound samples                   rom 8Kx8 , low rate

Except the 2 color palettes and sound samples every region has been
relocated to the external ram.

The pixel clock is 6MHz, the cpu clock is 3MHz.
 
The ram access is based on 16 slots scheme at 12MHz that is 8 pixels
duration. During that time we have :

 - 4 cpu access                      8bits
 - x sprite position                 8bits 
 - y sprite position                 8bits
 - background or sprite tile code    8bits 
 - background or sprite color        8bits
 - background or sprite tile graph1  8bits
 - background or sprite tile graph2  8bits
 - x/y/color big sprite              8bits 
 - big sprite tile code              8bits
 - big sprite tile graph1            8bits 
 - big sprite tile graph2            8bits 
 - 2 free slots                      -

There will be 1 cpu access every each 4 slots so the cpu can access
ram/rom without any wait state.

Background color contains 2 high bits of tile code.
Sprite color contains horizontal and vertical invert control  
 
x/y/color big sprite are 3 sequentialy accessed during the first 3 
sprites area.
  

Big sprite color contains horizontal and vertical invert control  

Video frame is 384 pixels x 264 lines.
  
Video display is 256 pixels x 240 lines.
Each lines contains 8 sprites and 32 background tiles. 
Each frames contains 28 background tiles height.

Each tile is 8x8 pixels
Each sprite is 16x16 pixels

Big sprite is a 8x8 tile graphic

Sound is composed of AY-3-8910 music and sound samples. 
--------------------------------------------------------------------

---------------
VHDL File list 
---------------

ckong_de2.vhd           Top level for de2 board  
ckong_de1.vhd           Top level for de1 board
ckong_de0_nano.vhd      Top level for de0-nano board

pll_50mhz_36mhz.vhd     PLL 36MHz from 50MHz altera mf

ckong.vhd               Main logic

video_gen.vhd           Video scheduler, syncs (h,v and composite)
line_doubler.vhd        Line doubler 15kHz -> 31kHz

ckong_sound.vhd         Music and samples logic
wm_8731_dac.vhd         DE1/DE2 audio dac

kbd_joystick.vhd        Keyboard key to player/coin input
ram_loader              Load external sram from fpga internal ram

rtl_T80/T80s.vhd        T80 Copyright (c) 2001-2002 Daniel Wallner (jesus@opencores.org)
rtl_T80/T80_Reg.vhd
rtl_T80/T80_Pack.vhd
rtl_T80/T80_MCode.vhd
rtl_T80/T80_ALU.vhd
rtl_T80/T80.vhd

io_ps2_keyboard.vhd      Copyright 2005-2008 by Peter Wendrich (pwsoft@syntiac.com)

ym_2149_linmix.vhd       Copyright (c) MikeJ - Jan 2005

----------------------
Quartus project files
----------------------
de2/ckong_de2.qsf             de2 settings (files,pins...) 
de2/ckong_de2.qpf             de2 project

de1/ckong_de1.qsf             de1 settings (files,pins...) 
de1/ckong_de1.qpf             de1 project

de0_nano/ckong_de0_nano.qsf   de0_nano settings (files,pins,...)
de0_nano/ckong_de0_nano.qpf   de0_nano project

-----------------------------
Required ROMs (Not included)
-----------------------------
You need the following 17 ROMs from ckongpt2.zip 
(MAME Crazy kong part II (set 1) - Falcon)

d05-07.bin
f05-08.bin
h05-09.bin
k05-10.bin
l05-11.bin
n05-12.bin

prom.v6
prom.u6
prom.t6

n11-06.bin
l11-05.bin
k11-04.bin
h11-03.bin

c11-02.bin
a11-01.bin

cc13j.bin
cc12j.bin

------
Tools 
------
You need to build vhdl or sram ROM image files from the binary file :
 - Unzip the roms file in the tools/ckong_unzip directory
 - Double click (execute) the script tools/make_ckong_proms.bat to get the following files

ckong_program.vhd
ckong_tile_bit0.vhd
ckong_tile_bit1.vhd
ckong_big_sprite_tile_bit0.vhd
ckong_big_sprite_tile_bit1.vhd
ckong_palette.vhd
ckong_big_sprite_palette.vhd
ckong_samples.vhd

ckong_sram_8bits.bin
ckong_sram_16bits.bin

*DO NOT REDISTRIBUTE THESE FILES*

Sram bin files are needed to load ROMs into the external sram 
VHDL files are needed if you want to recompile and include roms directly into the project 

The script make_ckong_proms uses make_vhdl_prom and and duplicate_byte executables delivered both in linux and windows version. The script itself is delivered only in windows version (.bat) but should be easily ported to linux.

Source code of make_vhdl_prom.c and and duplicate_byte.c is also delivered.

------------------------
Loading into de1 or de2
------------------------
3 steps 

 - program de1_usb_api.sof / de2_usb_api.sof (from Terasic CDROM) into the fpga

 - launch the Terasic de1/de2 control panel (Terasic CDROM)
     - use menu 'Open' to connect to the USB port
     - select SRAM tab
     - in the Sequential write enter : Address : 0, Length : 2A000
     - click the write a file to SRAM button and select ckong_sram_16bits.bin file
     - wait for write complete

 - program ckong_de1.sof / ckong_de2.sof into the fpga 

de0-nano has no SRAM built in so control panel cannot be used.

---------------------------------
Re compiling for de2 or de0-nano
---------------------------------
You can rebuild the project with ROM image embeded in the sof file. DO NOT REDISTRIBUTE THESE FILES.
4 steps

 - put the VHDL rom files into the project directory
 - activate the internal sram loader at the end of ckong.vhd file
 - rebuild ckong_de2 / ckong_de0_nano project
 - program ckong_de2.sof / ckong_de0_nano.sof into the fpga 

The rom data are transfered from fpga to the external sram at start-up.

de1 doesn't support sram loader as there is not enough fpga room to hold all the rom data.
de0-nano requires external hardware (128k sram, audio, video, keyboard interface)  

--------------------
Keyboard and swicth
--------------------
Use directional key to move, space to jump, F1 to start player 1 and F2 for coins.
de1 sw9 allow to switch 15kHz/31kHz
de2 sw0 allow to switch 15kHz/31kHz

for de0-nano interface see de0-nano top-level.

-------------------------------------------
Todo
-------------------------------------------
Use sdram instead of sram
Add ram loader from SD Card or other means

------------------------
End of file
------------------------
