# os|6.s081|interrupts

## top

- there are lots of memory is used as buffer(to reduce the times of operation) or cache(temporal and spatial locality)
- we usually don't have a lot of free memory so allocation needs to get back some memory firstly which leads to high overhead

## hardware

the hardware wanna catch the kernel

use trap to handle interrupt and get back

Differences:

- Asynchronous:no relationship between interrupt handler and CPU running current process
- Concurrency:devices runs dependently and should be mannaged wisely
- program device:devices actions

## where is it from?

- lines on motherboard connect devices with CPU
- in kernel va, we map va below 0x80000000 to devices
- All devices are connected together and interrupts are routed by PLIC(platform level interrupt control)
- it distributes every interrupts to some free CPU and save some metadata to trace the state of interrupt

Process:PLIC announce interrupt to all CPUS->a CPU heart will claim to receive->this CPU handle out and inform PLIC->no more save metadata

## driver

a section of code to manage device which is in kernel

- Bottom:interrupt handler to handle but not related to process context
- Top:API to call for this device or interact with user process
- Buffer:top and bottom write or read in or to it

Programe:we directly use pa to execute load/store, but unlike programming in user space, we only change the contents of controller registers to operate the device

- devices cowork with kernel: kernel is informed by interrupt when the device is ready

## xv6

shell output $ and input from keyboard:keyboard->UART->another UART->interrupt handler->screen

- SIE(supervisor interrupt enable):enable specific type of interrupt
- SSTATUS(SUPERVISOR STATUS):open or close all interrupts
- above two for every CPU heart
- SIP(supervisor interrupt pending):type of interrupt
- STVEC:save the current instruction in user process to return to

## uart-top

- Use file descriptor to present a device
- In UART, the size of buffer is 32 charts and a read pointer performs as a consumer and another write pointer as a producer
- if the device is free, read from buffer to THR which tells that we have a byte to be sent; when the data ends in device, syscall returns and shell continues

## uart-bottom

 when a CPU heart receive a interrupt:

- Clear SIE to close interrupt
- set SEPC as pc to return
- take down current mode and switch to supervisor mode
- Change pc into STECV(to start trap)

In usertrap, CPU heart tells PLIC to handle interrupt and get back interrupt number

a(only one) UART connects keyboard and console

## concurrency

- to promise automicity, we may close interrupt
- the top and bottom part of driver runs parallelly:different CPU heart may operate on buffer simultaneously but we use lock to control
- we call uartputc in top and uartintr in bottom, and the buffer is unique and in memory
- conditional synchronization:sleep with specific signal and wakeup when conditions meet by using same signal as parameter

## read from keyboard

- Console also has a buffer(128 charts) to read or write
- show commend:Keyboard->UART->console and store them in console buffer, and when it meets newline, it would wakeup shell to execute string in console buffer
- Use sleep to wait the other one

## polling

what if interrupt overturn the speed of interrupt handle?

- Keep CPU constantly asking whether are data in devices(controller registers), but if interrupt occurs less commonly, it wastes CPU sources
- to cut down the overhead of enter or exit interrupt, we use polling for frequent interrupts devices and interrupt for slow devices