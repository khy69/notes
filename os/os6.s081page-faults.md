# os|6.s081|page-fault

use error to enhance os!

a level of indirection:use va to access pa

Make pt dynamic, using which kernel could update pt

- previous handle:enter into kernel by trap and panic error va
- SCAUSE: distinct the reason why enter into kernel
- after trap, the original error position of instruction is stored in SEPC/trapframe->epc(in case that process switch), and we use this information to handle error
- STVAL:save the va where is trying to be accessed but errored

## lazy page allocation

- Allocate memory:user uses sbrk to enlarge its heap which is inited with 0
- Eager allocation:allocate specific size of physical memory; it may leads to ask for a large size but use a little
- allocate on use:old sz<va<current sz means that its a lazy allocation va in heap
- no physical memory leads to wisely handle or killing process

## zero fill on demand

- Text:instructions in program
- Data:inited global variables
- Bss:uninited or inited with 0 static or global variables
- If there are many variables equal 0 and all related pages in va could be maaped to **only a physical page** within 0; if we are trying to modify some contents in it, it is necessary to copy a new physical page
- and here, it assumes that we often use variables in a continuous page; and if we just use a few contents in all pages, it would cost a lot

## Copy on write fork

- previous fork would copy all pages for child process from parent process; but some times, e.g execute exec in child, we would replace these pages rapidly
- a lazy manner:only map va of child process to pa of parent process(only for read)
- if child try to wirte to a read only page, a page fault  would help to copy a physical page and remap this va of child, and child could write here
- not all read only page is considered as a cow page, so we are supposed to use RSW in PTE to identify cow scenes
- trampoline is the same for all users which is mapped from every user to a same pa
- to release physical page in a proper time, we need to count how many va is mapped to this

## demand paging

- only allocate va papge for text and data and no actual pa page
- process execute from the instructions residing va 0
- Need->read into memory->map va to pa->execute
- Dirty page:have been writen but not to file yet;like cache:written but not to memory
- we could reset access bit timely to evict page using LRU

## memory mapped files

- load a partial of file into va and access tem by va and write back dirty block when va unmapped
- lazy method:for fd, VMA(virtual memory area) save metadata to find the corresponding va for file and load when occurring page fault in this range