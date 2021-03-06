Anatol
------
Anatol is a very simple breadboard computer based on the W65C02S 8-bit processor. It's really not much different from all other 6502 breadboard computer projects, and is *heavily* inspired by Ben Eater's video series on the subject. However, this project has a much better name than other 6502 computers, justifying its existence.

Software
--------
I tried to implement some software to simplify my life in building this computer. In the current setting, I have an Arduino Mega connected to the computer's data bus, address bus and to some pins of each chip in order to be able to monitor and control what is happening. The clock signal is also generated by the Arduino board at a software-defined rate.

The code running on the Arduino is actually only a command server that listens on serial for commands to be executed. Currently the commands I have are:
+ Write Clock
+ Read/Write address bus
+ Read/Write data bus
+ Read/Write EEPROM

All of these commands are implemented in a way that no two chips will be writing on the buses at the same time. But, it doesn't check much the arguments that are provided and will be happy to do something stupid if asked to.

The Arduino command server is controlled by a Python client. At the moment, the python client is simply a Python class that I use from a REPL to do whatever I need to do. It implements higher level functionalities on top of the Arduino's capacities. Currently, the Python client can:
+ Run the clock and print the data and address bus at every clock cycle
+ Assemble and upload programs to the RAM
+ Simulate the memory circuit by monitoring writes to the VRAM. Each time the VRAM is written to, the simulated screen will by printed in the console.

Examples:
       + Initialze
              > interface = Interface(clock_speed=500) # run at 2hz!
              > interface.connect("/dev/tty.usbmodem144101")
       
       + Read reset vector
              > interface.rom_read(0xfffc, 2)

       + Write nop loop manually (thankfully, I don't need to do this anymore)
              > interface.rom_write(0x8000, [0xea, 0x4c, 0x0, 0x80, 0xea])

       + Compile and upload assembly program
              > interface.upload_program("programs/hello_world.s")

       + Run the computer (Ctrl+C to interrupt)
              > interface.clock_run()

       + Run single clock step
              > interface.clock_tick()

I think I still have a speed problem with this setup, making it impossible to run fast. I didn't investigate this yet, but I will be able to get rid of a lot of software as soon as I have a built version of the graphics hardware and keyboard.

Chips I use
-----------
+ W65C02S, the processor
+ AT28C256, the EEPROM
+ AS6C62256, the RAM

Memory map
----------

0x0000 +----------------------
       | Zero page
0x0100 +----------------------
       | Stack 
0x0300 +----------------------
       |
  ...  | Free
       |
0x7C40 +----------------------
       | TXT VRAM (960B)
0x8000 +----------------------
       |
  ...  | ROM
       |
0xFFFA | NMIB (LO) -- Non-maskable interrupt
0xFFFB | NMIB (HI)
0xFFFC | RESB (LO) -- Reset vector
0xFFFD | RESB (HI)
0xFFFE | BRK/IRQ (LO) -- Break / Interrupt Req
0xFFFF | BRK/IRQ (HI)
       +----------------------

VRAM size
---------
screen = 640x480 pixels 
1 char = 20x16
=> 40x24 chars = 960 chars = 960B needed for VRAM