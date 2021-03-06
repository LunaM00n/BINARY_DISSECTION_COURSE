#	INTRODUCTION TO THE WORLD OF BINARIES
Every program (a compiled binary) is just a combination of code and data. The code which are low level instructions deal with the data to produce some output. Since the code and data are embedded inside a single binary file, there should be a way for the machine to distinguish between them and the inability of the machine to do so may result in disastrous effects. This is indeed the root cause of *code injection attacks*.<br>

When a source code is compiled, the machine code generated is termed as 'object code' and may be stored in a file. Linker combines one or more of such object code files to produce a final executable binary. Object code generated is one of the blocks of the final executable which may be made up of many such blocks. <br>

For a machine to execute the binary, it must know in advance how the code and data in the file is laid out, i.e. it needs to know the format of the binary. Two of the most common executable formats are:
* **E**xecutable and **L**inkable **F**romat (**ELF**) for Linux
* **P**ortable **E**executable (**PE**) for Windows (Extensions - `.exe`, `.dll`, `.sys`, `.ocx`, `.cpl`)
* **Mach-O** for OSX.

For this course we will be focusing on the ELF executable format and maybe later we'll cover Mach-O and PE file format too.



## A SIMPLE ELF BINARY
Let's look at the simple binary file `main` compiled from source code present in this directory [main.c]. We'll use the tool `hexdump` to see the raw binary file.<br> Hexdump is a command line utility which formats the user supplied input files into a user specified format.

  ```bash
critical@d3ad:~/DISECTING_BINARIES_COURSE/Introduction$ echo "int main() {}" > main.c
critical@d3ad:~/DISECTING_BINARIES_COURSE/Introduction$ cat main.c 
int main() {}
critical@d3ad:~/DISECTING_BINARIES_COURSE/Introduction$ gcc main.c -o main
critical@d3ad:~/DISECTING_BINARIES_COURSE/Introduction$ hexdump main
0000000 457f 464c 0102 0001 0000 0000 0000 0000
0000010 0003 003e 0001 0000 04f0 0000 0000 0000
0000020 0040 0000 0000 0000 18e0 0000 0000 0000
0000030 0000 0000 0040 0038 0009 0040 001c 001b
0000040 0006 0000 0004 0000 0040 0000 0000 0000
0000050 0040 0000 0000 0000 0040 0000 0000 0000
...
```
 

## ANALYSING HEXDUMP
Each output line constitutes of 16 bytes of the file.
```bash
0000000 457f 464c 0102 0001 0000 0000 0000 0000
0000010 0003 003e 0001 0000 04f0 0000 0000 0000
```
A binary file is usually represented in hexadecimal format since a series of 0's and 1's probably don't make much sense to us humans. 
 * **FIRST LINE** : The first of group of bytes `0000000` is the offset to the first byte, i.e. 0x7f (not 0x45 - due to little endian byte order) in file. FILE OFFSET here means the distance(in bytes) of first byte in line from first byte of the file, which accounts to 0 since the first byte of this line itself is the first byte of the file. After this there are 8 groups (of 2 bytes each) which accounts for 16 bytes in the file. Sincde.
 * **SECOND LINE** : The offset to the first byte in this line is `0x0000010` (i.e. 16 bytes in decimal), therefore `0x03` (not `0x00` - due to little endian byte order) is 16 bytes away from the first byte in the file. This makes sense as the each line of hexdump displays 16 bytes.

 **LITTLE ENDIAN byte order** : It specifies the order in which the data bytes are stored in memory. Little Endian byte ordering explains that the least significant byte will be stored first and then the more significant bytes.

  **BIG ENDIAN byte order** : It specifies that the most significant byte is stored first and then the lower significant bytes.

For Example : In the number `169`, the least significant digit is 9 (at 1's place), then 6 (at 10's place) and the digit 1 here has the most significant place (100's), therefore the most significant digit. Similarly, suppose the hexadecimal number `0x080400ab` in memory will be stored in little endian byte order i.e. - `ab 00 04 08`. Intel CPU's are little endian and will interpret this value `0x080400ab` stored in memory as `0xab000408` (i.e. in revere order).

**NOTE** : These hexadecimal bytes would make more sense ahead.


##  KNOWING THE FILE TYPE
Now that you know how to reach out to any offset/location in a binary and examine content, let's move furthur and understand the *file type* of this binary.

```bash
critical@d3ad:~/COURSE_DISECTING_BINARIES/Introduction$ file main
main: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=0d072f7bfdfee55c728743f9ac6e7e08309ed581, not stripped
```

Let's see what all this information means
* `main` : It is the name of the binary
* `ELF 64-bit LSB shared object` : It is a 64-bit compatible shared object file. Executable files and shared object files are similar to each other, having a few differences which I'll explain ahead. LSB says that it is encoded in 2's complement, little endian format.
* `x86-64` : It is compiled to run on a 64-bit platform (processor architecture). 
* `version 1 (SYSV)` : It is some field set to 0 in ELF header. (Not relevant right now)
* `dynamically linked` : Tells how the linking is done (Static/Dynamic). Will go through linking later on.
* `interpreter /lib64/ld-linux-x86-64.so.2` : Specifies location of the linker.
* `GNU/Linux 3.2.0` : The kernel version for which the C library is compiled.
* ` BuildID[sha1]=0d072f7bfdfee55c728743f9ac6e7e08309ed581` : Identifier for build session producing the binary. Useful while debugging.
* `not stripped` : Stripping a binary removes all the information that is useless to the binary to execute. Stripping makes the binary difficult to reverse engineer or debug.   

Next, we'll move on to understanding the ELF's.
Don't worry if any/some of the above fields does'nt make any sense. Just move ahead and trust me you'll get more comfortable :)
  


<br>

**[PREV - COURSE CONTENT]** <br>
**[NEXT - ELF]**


[PREV - COURSE CONTENT]: ./../README.md
[NEXT - ELF]: ./../ELF/ELF.md
[main.c]: ./main.c
