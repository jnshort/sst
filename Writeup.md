Author: Justin Short  
Date: 3/24/2025  
Class: CECS 478  

# Lab 4: Malware
This lab involves examining and altering a binary file containing the game "Super Star Trek". The binary file has been modified from the original verison of the game, removing the ability for the player to set the self destruct password.  
The goals of this lab are to:
1) Analyze the binary in order to find the password
2) Patch the binary in order to bypass the password check
3) Patch the binary in order to add my name to the intro credits

## Analyzing the Binary
This binary file is a compiled C program. I chose to use Ghidra to explore and edit the binary file. In previous assignments (378 malware lab), I utilized a generic hex editor to accomplish this (due to finding ghidra overwhelming to use). However, since I was going into this project knowing what general steps I would need to take, I decided to try using Ghidra.  
Loading the binary file into Ghidra, I was able to decompile it into assembly instructions. I knew, since I was trying to bypass a password check, that I was looking for somewhere a user input was compared against a known string. I found that there were 3 instructions in the binary that made system calls to "strcmp".  


## Finding the Password
In order to determine which of these instructions was the password check, I started the game and initiated the self destruct sequence. Each line that is printed to the console is stored in the data section of the binary file. By finding the functions that print out these strings, I was able to determine that the call to "strcmp" at memory location 0x004037d6 was the password check I needed to bypass. I also was able to determine that the password that the player input was compared against was stored at memory location 0x0041f628. However, this value is uninitialized, meaning it is set at some point between when the program is started and when the password check occurs.  
![strcmp instruction](/photos/strcmp_before.png)  
To determine what the password is set to, I ran the binary file from inside of gdb. I initiated the self destruct, then interupted the program when it asked for the player to input the password. I was then able to examine the value at memory location 0x0041f628, and found the password to be "dedchrme".  
![password in memory](/photos/password_in_memory.png)  


## Bypassing the Password
Know that I knew where the password check was done, bypassing the check was very straightforward. I considered two ways of bypassing the password. The first method was to add a jump instruction just before the user is prompted for input, and jump to the code that executes after the password is accepted. The second way was to find a way to force the strcmp to always evaluate to "True", which was the method I decided to implement.  
The strcmp call functions by loading a pointer to the first argument into register EDI and a pointer to the second argument into register ESI. Then when the strcmp call is made, the string values at each register will be compared.  
In the binary file, the pointer to the password is loaded into ESI, then the pointer to the user input is loaded into EDI. I patched this second instruction, which loaded the EDI register. Instead of loading it with a pointer to the user input, I loaded it with the value that was already stored in ESI. This meant that ESI and EDI now both contained pointers to the same memory location, and by extension, the strcmp function would always evaluate to true. Since the instruction I edited it to was shorter in length than the original (2 bytes instead of 5), I also added 3 NOP instructions to make up the difference.
![patched instruction](/photos/strcmp_hacked.png)  

The equivalent C code would look like:  
![patched instruction C](/photos/equivalent_C.png)  

## Adding My Name to Credits
When the program needs to write to the console, it will load a pointer to a string into the EDI register, then it call a function to print to the string to the console. If the program wants to print out an empty line, it will load an integer of how many line to skip into the EDI register, then call a skip function to print that many empty lines. Below is the original instructions.  
![original instruction](/photos/credits_before.png)  
My goal was to add my name to the credits between the "CECS 478 Edition" line and the "*Hacked* by anthonyg" line. In order to print a custom string, I needed to store the string I wanted in memory. I found a section of the binary that was all "0" and seemed to only serve the purpose of padding.  
![padding](/photos/store_data_empty_space.png)  
I added a null terminated character array (string) to this section that contained the string "*Hacked* by Justin Short".  
![added string](/photos/added_string.png)  
Now all I needed to load the pointer to this string into the EDI register and then call the function that prints the string to the console.  
![print custom string](/photos/credits_hacked.png)  

## Results/Deliverables
Hacked Intro Credits:
![credits](/photos/credits.png)  

Password Bypass(Accepted regardless of what the user inputs):
![destruct bypassed](/photos/destruct_bypassed.png)  
