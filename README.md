 MiniOS – A 32-bit Operating System Kernel 

Project Guide & Documentation 

 1. Introduction 

MiniOS is a simple yet fully functional 32-bit operating system kernel developed using: 

    C Programming 

    x86 Assembly 

    GRUB Multiboot Bootloader 

    GCC Cross Compiler (i386-elf) 

    QEMU Virtual Machine 

The goal of this project is to understand core Operating System concepts by building the kernel from scratch, without Linux or any existing OS libraries. 

MiniOS demonstrates: 

    Hardware-level keyboard input 

    Direct writing to VGA text buffer 

    A shell interface 

    Arithmetic expression evaluation 

    Viewing embedded files 

    Shutting down QEMU from inside the OS 

This project gives real hands-on experience in systems programming and OS development. 

 

 2. Project Overview 

MiniOS boots using the GRUB Multiboot specification, which loads the kernel into memory and jumps directly to the entry point defined by the OS. 

Once the kernel starts, it: 

    Initializes the VGA text mode display 

    Initializes keyboard input 

    Prints a welcome message 

    Starts a shell loop 

    Accepts and executes commands 

MiniOS is a bare-metal kernel, which means it runs without any operating system underneath. It interacts directly with hardware using I/O ports and memory-mapped video memory. 


3. Boot Process Explained 

3.1 CPU Starts in Real Mode 

    QEMU’s BIOS runs in 16-bit real mode 

    BIOS loads GRUB Stage 1 from the ISO 

3.2 GRUB Loads Stage 2 

    GRUB can read filesystems (ISO9660) 

    Reads the file /boot/kernel.bin 

3.3 GRUB Detects Multiboot Header 

Your kernel contains: 

0x1BADB002 
This tells GRUB: 

    Kernel is Multiboot-compliant 

    Load at the memory address defined in linker.ld 

    Jump to _start 

3.4 GRUB Switches to Protected Mode 

GRUB enters 32-bit protected mode before calling your kernel. 

 This avoids writing complicated real-mode assembly. 

3.5 GRUB Calls _start() 

Execution enters your kernel and then transitions into: 

kernel_main(); 
  

 

 4. File Structure 

minios/ 
├── src/ 
│    ├── kernel.c      → The kernel code 
│    └── message.S     → Embeds message.txt 
├── message.txt        → Text file displayed by OS 
├── grub/ 
│    └── grub.cfg      → Bootloader configuration 
├── linker.ld          → Memory layout script 
├── Makefile           → Build system 
└── minios.iso         → Bootable ISO output 
  

5. Important Components Explained 

5.1 Linker Script (linker.ld) 

Controls where the kernel is loaded in memory: 

ENTRY(_start) 
SECTIONS { 
    . = 1M;     /* Start kernel at 1MB */ 
    .text  : { *(.text*)  } 
    .data  : { *(.data*)  } 
    .rodata: { *(.rodata*)} 
    .bss   : { *(.bss*)   } 
} 

5.2 VGA Text Mode Output 

Video memory starts at 0xB8000. 

You write characters like this: 

v[(row * 80 + col) * 2] = 'A'; 
v[(row * 80 + col) * 2 + 1] = 0x0F;  // white on black 
This is how your OS prints text. 

5.3 Keyboard Driver 

Keyboard scancodes come from I/O port 0x60. 

unsigned char sc = inb(0x60); 
Shift and normal keys are decoded separately. 

 

6. Arithmetic Expression Engine 
Example: 

4*4+9*9 → 97 
3+4*5  → 23 
This teaches compiler design concepts. 

7. Embedded File Viewer 

message.txt is embedded inside the kernel using: 

.incbin "message.txt" 

The kernel can print it when running view. 

This shows how OSes embed firmware blobs. 

8. Shell Commands 

MiniOS supports: 

Command 
	

Description 

help 
	

Show command list 

clear 
	

Clear VGA screen 

echo text 
	

Print text 

view 
	

Show message.txt 

quit 
	

Shutdown QEMU 

1+1, 4*4+9*9 
	

Math expressions 

The shell works in a loop: 

    Read keyboard input 

    Build command string 

    Parse command 

    Execute action 

    Print result 

9. Build System (Makefile) 

Your Makefile automates: 

    Compiling kernel 

    Linking kernel 

    Creating ISO 

    Running QEMU 

ISO is created using: 

grub-mkrescue -o minios.iso iso/ 
  

This ISO is bootable on real hardware. 

10. Running MiniOS 

make clean 
make 
make run 
  

You see: 

MiniOS Loaded  
Commands: help, view, echo, clear, quit, math like 1+1 or 4*4+9*9 
> 

11.Output 

12. Future Improvements  

    Add ls and cat with a mini filesystem 

    Add timer interrupts (clock tick) 

    Implement a memory allocator (malloc) 

    Add graphics mode (VGA) 

    Add multi-tasking 

    Add file system support (FAT32)