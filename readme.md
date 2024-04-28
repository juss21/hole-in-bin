# Hole-in-bin

## ex00

First we need to get the specifics of the main function. I will do it in every following exercise.
We can do it like this:     
```bash
# Assuming you're in the exercise folder
gdb ./bin     
disas main
```    
<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/00_main2.png" max-width="400" max-heigth="400" />   

As we can see the application uses `gets()` function. It can be exploited, because `gets()` doesn't check for buffer overflows.   

We can calculate the buffer size by subtracting (%eax) buffer offset address from end address of the buffer `0x5c - 0x1c = 0x40`    
So our input needs to be longer than 0x40 (64 in decimal) characters.  
![pic1](https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/00_done.png)  

## ex01 

<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/01_done.png" max-width="400" max-heigth="400" />  

The program compares input/argument to 0x61626364 (`abcd` based on ascii table from hex value).     
It expects the input string to be in little-endian format, meaning that the least significant byte is stored first in memory.   
This time end of the buffer address is `0x60` and start `0x1c` so the argument needs to be `0x44` (68 decimals) long.   

As the least significant byte is stored first, we need to write out `abcd` in reverse: `0000000000000000000000000000000000000000000000000000000000000000dcba`

## ex02

<img src="https://01.kood.tech/git/juss/hole-in-bin/raw/branch/master/images/02_done.png" max-width="400" max-heigth="400" />     

This was pretty much copy-paste from the previous exercise.     
Only in this one you had to give a correct value to a variable before running the application.  

Hex value you need to give to your variable in order to pass the comparison:    
`d0a0d0a` which translates to `\r\n\r\n`.  
Because the task expects little-endian format again, we need to reverse the variable:   
`GREENIE=$'0000000000000000000000000000000000000000000000000000000000000000\n\r\n\r'`