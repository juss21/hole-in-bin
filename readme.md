# Hole-in-bin

## ex00

First we need to get the specifics of the main function.
![pic]()

We can determine if your input length fits inside the buffer with this calculation:     
`0x60 - 0x5c >= (%esp - 0x60) - (0x1c - %esp)`

- `0x60 - 0x5c` - This calculates the size of the buffer by subtracting the start address of the buffer (0x5c) from the end of the allocated space (0x60)

- `(%esp - 0x60) - (0x1c - %esp)` - This calculates the size of the input by subtracting the start address of the buffer from the end of the allocated space.