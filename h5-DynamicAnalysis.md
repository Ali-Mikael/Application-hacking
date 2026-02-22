# "It works on my machine"
<img width="1473" height="848" alt="Screenshot from 2026-02-10 16-23-21" src="https://github.com/user-attachments/assets/e41794df-ec24-48fa-9001-4127d797f876" />

-----


# A) LAB_1
**Objective**
- Investigate what's wrong with the [program](<https://terokarvinen.com/application-hacking/lab1.zip>) and how to fix it


### First things first
Unzip the target, remove zip file and move into the directory
```bash
$ unzip lab1.zip && rm lab1.zip && cd lab1
```

**We're left with**
- The src code
- The binary
- Build instructions
- <img width="847" height="91" alt="Screenshot from 2026-02-09 21-01-05" src="https://github.com/user-attachments/assets/c5e65d37-f95c-4bae-98ff-206cac07f146" />


We start off by doing a quick sanity check:
- <img width="1478" height="159" alt="Screenshot from 2026-02-10 18-15-22" src="https://github.com/user-attachments/assets/043321ef-217a-42cd-92d8-46f3f1a98f93" />

Run the program and see what happens:
- <img width="1478" height="238" alt="Screenshot from 2026-02-10 18-16-48" src="https://github.com/user-attachments/assets/494947d1-dee6-4214-9231-3f1b06e31ad3" />

We learn that the program is causing a segmentation fault. So we investigate ->


### Debugging
We start the _GNU Debugger_ by issuing the command:
```bash
$ gdb gdb_example1
```


We want to see the `C` code being executed side by side, so we enable the _text user interface_:
```bash
$ (gdb) tui enable
```
- You can also skip this step by initially starting GDB with the `-tui` parameter

On the prompt, we issue commands:
```bash
(gdb) break 5 
(gdb) break 14 
```
- This will effectively set breakpoints at the **first line** inside the `print_scrambled()` and `main()` **functions**
- We can also do the abbreviated version: `b <line>`
- <img width="1533" height="826" alt="Screenshot from 2026-02-11 11-45-53" src="https://github.com/user-attachments/assets/724dd9f6-65c7-43cb-b580-f7a2305048b7" />


We **run the program** and it automatically stops at the first break point inside `main()`
```bash
(gdb) run
```
- <img width="931" height="179" alt="Screenshot from 2026-02-12 08-25-31" src="https://github.com/user-attachments/assets/eee898ef-e0c5-44b8-b603-ac84fc8ba199" />

### Executing the code
> [!TIP]
>
> **Execution control commands**
> - `next`: _Execute next line, **step over** functions_
> - `next x`: _Execute X lines of code_
> - `step`: _Execute next line, **step into** functions_
> - `step x`: _Execute X lines of code, step into functions_
> - `continue`: _Resume execution until next breakpoint_
> - `finish`: _Run until current function returns_
> - `until`: _Continue until a certain line or loop ends_
>
> **Note also** that you can abbreviate the commands as long as there is no ambiguity!
>
> Meaning if you have two commands: `continue` and `complete`, you have to type `con` or `com`


**Getting information on the local variables:**
- <img width="976" height="128" alt="Screenshot from 2026-02-12 08-25-16" src="https://github.com/user-attachments/assets/28aeb5f9-93fd-4da7-931a-28193ba1f6a9" />
- `i lo` == `info locals`


We then **step forward one line** and inspect the variables again
- <img width="721" height="130" alt="Screenshot from 2026-02-12 08-30-43" src="https://github.com/user-attachments/assets/193fdd84-1013-468c-8361-2162be8f4054" />


Once we get to `line 17` where a function call is made, we use the `step` command
- We receive some information on what's going on:
  - <img width="1648" height="125" alt="Screenshot from 2026-02-12 08-45-47" src="https://github.com/user-attachments/assets/8658b0d8-00db-4ebc-a2ba-32bdf4597fad" />
  - <img width="851" height="363" alt="Screenshot from 2026-02-12 08-45-32" src="https://github.com/user-attachments/assets/68ca5aa5-6132-451a-90d8-b782cbba05d9" />


Instead of manually typing `print` or `i lo` every time, we want GDB to display them for us automatically.

So we issue the command:
```bash
(gdb) display message
```
This will show us the value after each step
- <img width="744" height="399" alt="Screenshot from 2026-02-12 08-58-06" src="https://github.com/user-attachments/assets/4b1276ea-283a-4674-a728-f0e034b53991" />

We skip to the end of the loop with the `until` command and see the scrambled message:
- <img width="564" height="202" alt="Screenshot from 2026-02-12 09-00-55" src="https://github.com/user-attachments/assets/07a43b07-1687-464d-956d-e2b8d57627a8" />


But what happens when we try to scramble a variable with the value of `null`?
- The code passes the variable to the function as an argument
  - `print_scrambled(bad_message)`
  - <img width="1179" height="163" alt="Screenshot from 2026-02-12 09-01-29" src="https://github.com/user-attachments/assets/e3500f1e-3f13-4678-8cfd-bd61099380a2" />
- What follows is a **segmentation fault**:
  - <img width="1310" height="171" alt="Screenshot from 2026-02-12 09-02-04" src="https://github.com/user-attachments/assets/8a4e42d5-8b58-4b61-b9be-b3821f30595d" />
  - <img width="1100" height="88" alt="Screenshot from 2026-02-12 09-02-35" src="https://github.com/user-attachments/assets/1e5812eb-b100-40b6-a592-18e4cdd77e73" />



## What is a segmentation fault?
**Simply put:**
- _The program tries to access a memory location without having the permission to do so_
  - The hardware will sense this and execute a jump to the OS
  - ON UNIX family platforms the OS will normally _announce that the program has caused a seg fault and stop the execution of the program_

(Norman M. Peter Jay S, 2008)


### Fixing it
The problem:
- The program is trying to access a memory location it has no **access rights** to
- Why? Because the variable is set to `null`


So we simply change the value to **not null**
- I'm doing it straight from the source code, as we still have access to it!
- <img width="646" height="132" alt="Screenshot from 2026-02-12 11-58-34" src="https://github.com/user-attachments/assets/c089c789-35b2-4f56-8587-f1c3e3f1914e" />


Recocompile:
```bash
$ gcc -g -o example1 gdb_example1.c
```
By running the recompiled program **example1** alongside the **faulty one**, we can see that we got rid off the _seg fault_:
- <img width="1091" height="242" alt="Screenshot from 2026-02-12 12-02-02" src="https://github.com/user-attachments/assets/2cb3afb3-a7c3-4634-a574-8d6c5f0f052c" />



**Help received**
- Norman M, Peter Jay S. 2008. [The Art of Debugging With GDB, DDD and Eclipse](<https://zhjwpku.com/assets/pdf/books/The.Art.of.Debugging.with.GDB.DDD.and.Eclipse.pdf>)
- [The GDB developer's GNU Debugger tutorial, Part 1: Getting started with the debugger](<https://developers.redhat.com/articles/the-gdb-developers-gnu-debugger-tutorial-part-1-getting-started-with-the-debugger#>)
- [GDB commands](<https://www.tutorialspoint.com/gnu_debugger/gdb_commands.htm>)
- [Quick Guide to gdb: The GNU Debugger](<https://kauffman77.github.io/tutorials/gdb.html>)


-----

# B) LAB_2
**Objective**
- Crack the `passtr2o` program
- Find out the password and flag
- Write a report on how it opened


> [!NOTE]
> The execution environment has changed. New laptop, new OS:
> - <img width="1037" height="362" alt="2026-02-21-18:50:26" src="https://github.com/user-attachments/assets/36a5dd75-c102-420b-aaa3-58a623556ff7" />
> 
> Now that that's out of the way, let's continue!


We have the [target](<https://terokarvinen.com/application-hacking/lab2.zip>) downloaded and unzipped, let's get to work.


In the target directory we have the old `passtr` program, and the new one: `passtr2o`. 

We're going to be focusing on the latter one!


### Poking around
We naturally start by executing the program
- <img width="576" height="120" alt="2026-02-21-18:56:53" src="https://github.com/user-attachments/assets/aa778fd8-75b8-4602-bb08-8bd144268ec0" />

Then a quick check for any low hanging fruit:
- <img width="689" height="641" alt="2026-02-21-21:10:12" src="https://github.com/user-attachments/assets/81078435-eaf8-4ff3-a112-6b35e3515408" />

I noticed the string `anLTj4u8`, but at least in the current format doesn't give us the flag :/
- <img width="563" height="81" alt="2026-02-22-01:14:07" src="https://github.com/user-attachments/assets/20cdffcd-64f6-4f94-a48a-f668a1898b9e" />


We then quickly go through the ASM using the `objdump` utility.

**What is objdump?**
- Straight from the man pages: _display information from object files_
- The options for this utility are many, but we stick to the `-d` flag, which will `disassemble` it for us
  - <img width="1049" height="416" alt="2026-02-21-21:27:05" src="https://github.com/user-attachments/assets/a31fa4ac-d545-4212-8410-e1e19a5052b9" />

We navigate to the main function:
- <img width="1050" height="873" alt="2026-02-21-21:27:47" src="https://github.com/user-attachments/assets/01a6a017-f2ac-4147-8f2d-5c70e10c0266" />


Highlighted in **red**: we can see that after prompting the user for input, the input is passed on to the `mAsdf3e`-function, which most probably takes it and does the string comparison against the password (among other things).

Highlighted in **green** we notice that if strings do **not** match, there's and instruction following it, `jne` (jump not equal), which will **jump over** the `EaseAs` function. Which I suspect contains the flag!
  - <img width="727" height="194" alt="2026-02-21-22:01:00" src="https://github.com/user-attachments/assets/b6f61050-1a5d-40b8-8205-a45de3438c82" />

I said earlier "among other things", because the `mAsdf3a` function is not exactly short:
  - <img width="718" height="643" alt="2026-02-21-21:57:33" src="https://github.com/user-attachments/assets/5c678e9e-134a-40e4-aa0d-f0f37b02af53" />


For further examination, we're going to have to open up the GDB -->


### One step at a time
We start the gnu debugger in `TUI` mode and give the command
```bash
layout asm
```
This way we'll see the instructions above the prompt at all times.
- <img width="1896" height="1098" alt="2026-02-22-03:44:02" src="https://github.com/user-attachments/assets/e97c1c54-1d30-4328-aab6-8743adee4213" />


We then set the breakpoint at `main` and run the program:
```bash
(gdb) b main
(gdb) run
```



### Workflow
We use commands `si` & `ni` to go through the program. These are the `step/next` equivalents for individual instructions.

Disclaimer:
  - I did a whole bunch of stuff which I unfortunately didn't have time to report on, so I'm skipping the "messing about" part where i'm just trying to figure out what's going on.

I ended up exiting and starting gdb again, this times setting the break point at the start of the `mAsdf3a` function. When prompted for the password I entered in the letter `e`. 

I went forward one line at a time and finally hit a **compare instruction**
  - <img width="512" height="37" alt="2026-02-22-00:39:59" src="https://github.com/user-attachments/assets/5ce35a65-b57d-44f7-afb6-4cbfeb927358" />

We naturally printed the values of the targets
  - <img width="184" height="81" alt="2026-02-22-00:40:12" src="https://github.com/user-attachments/assets/1c4886fa-3369-413e-82ac-56dad2a425fe" />

We can see that the values are not equal, and next up we have a `jump not equal` instruction, which then jumps over the section marked in red
  - <img width="812" height="438" alt="2026-02-22-00:43:15" src="https://github.com/user-attachments/assets/58ef8bb5-1c00-4903-8e8e-296fa3fa1565" />

I wanted to know what's going on inside, so I did some more digging. 

Started again from the same breakpoint, but this time input `22` as the password when prompted.

Now when we hit the compare instruction and compare the targets, instead of 1, my value is 2. But the `$r12d` is still 8.
  - <img width="172" height="78" alt="2026-02-22-00:54:56" src="https://github.com/user-attachments/assets/c33f1be2-2975-4d55-8f63-79520336be25" />

This made me suspect that the password string is 8 characters long.


So I went back to the compare instruction and set the register `$edx` to `8` by using the command
```asm
set  $edx = 8
```
This way we **don't jump**, and resume execution at the next instruction on the list. So it confirms our doubts.


At the same time of realising this, I had an idea: 

The string I caught earlier was 8 characters long as well, so I decided to pass it to the function as an argument and automatically pass the first check!

I then noticed that the program subtracts `7` from some value, does some operation, and then compares it with user input.
```asm
sub  $0x7,%edx
```

I printed the value stored in `%edx` and got = `97`
- <img width="132" height="45" alt="2026-02-22-01:44:33" src="https://github.com/user-attachments/assets/6e117fad-4deb-497e-924b-b77f74b8823a" />


A quick browser hit tells us that `97` in ascii is = `a`

Then to the actual comparison:
```asm
cmp  %ecx,%edx
```
I printed each value being compared:
- <img width="155" height="82" alt="2026-02-22-01:48:20" src="https://github.com/user-attachments/assets/c6aa4973-9d53-4eba-a930-82c74f204983" />

And made my deductions:
- ascii value 97 represented as a letter is `a`
- ascii value 100 represented as a letter is `d`

I deduced that the password starts with the letter `d`, and right now there seems to be an offset of 3 between the string we found and the correct password.

The comparison wasn't equal, so it jumped.

No worries, we run it again at the same breakpoint, and this time take the string `anLTj4u8` _shifted forward by 3 ascii_ and give it to the program:
- <img width="735" height="201" alt="2026-02-22-01:58:00" src="https://github.com/user-attachments/assets/306ebe7a-a178-4478-bd38-a810cfb6a246" />

Now when the `comparison` happens again, the values are equal
```
%ecx = 100
%edx = 100
```

We're still in the game and get to see another round. 

By moving forward we got to a `jmp` instruction, where some magic happens. 

The following caught my interest again
```asm
sub  $0x7,%edx
```

I print the value of `%edx` (For clarity: this is before the subtraction has taken place)
```asm
(gdb) print %edx
```
- This time around it's = `110`
  - `110 - 7 = 103`
  -  The ascii value 103 represent the letter = `g`

Our letter unfortunately doesn't match, so the at the next compare operation it fails and jumps. 

So I change my passwords second letter from `q` to `g` and run the program again.

At this point it felt like i'm brute forcing the password and there has to be a better way of doing this. And that's when I noticed the pattern!
Just as it felt like i'm brute forcing the password, and there has to be a better way of doing this, I noticed a pattern
- <img width="391" height="96" alt="2026-02-22-02:17:43" src="https://github.com/user-attachments/assets/94555983-9ca9-41c6-96aa-c78f4effb9bd" />

First first the program _either_ **subtracts 7** from `%edx`, _or_ **adds 3**, and then **compares it against** our value stored in `%ecx`.

### The aha moment
Now on the _second round_ when the _second letter_ is `g`, the comparison is equal
- <img width="150" height="84" alt="2026-02-22-02:24:26" src="https://github.com/user-attachments/assets/be03ac4a-a1ec-4620-ae58-8e6829b5f862" />

We move forward in the instructions, and the value 3 is added to the value stored in `%edx`
```asm
add  $0x3,%edx
```

We print the value
- <img width="145" height="44" alt="2026-02-22-02:28:36" src="https://github.com/user-attachments/assets/251dc480-2133-428a-9748-33a565ceca8b" />
- 76 + 3 = 79
- 79 in ascii = O

The patterns seems clear now, first round the value 3 is added, the next round 7 is subtracted, and then it goes back to 3 etc....

If we take the string we found earlier and apply the logic

| a | n | L | T | j | 4 | u | 8 |
| - | - | - | - | - | - | - | - |
| +3 | -7 | +3 | -7 | +3 | -7 | +3 |
| d | g | O | M | m | - | x | 1 |


We are left with the flag:

<img width="705" height="82" alt="2026-02-22-02:42:28" src="https://github.com/user-attachments/assets/5c3ea953-3f2c-46fd-9a16-17bc19abc2f5" />


I still wanted to check something out, and `ghidra` confirmed my doubts
- If the value is **even**, 3 is added
- If the value is **odd**, 7 is subtracted
- And by value I mean the `index` of the char array.

Here we can see it in "plaintext"
- <img width="505" height="518" alt="2026-02-22-02:37:35" src="https://github.com/user-attachments/assets/43de147c-fefc-44fd-abf9-5e69cf18e639" />
- ( I didn't bother to clean up the pseudocode just for a quick check.. )

### What's going on?
- The program is doing a simple bitwise operation using the `AND` operator
- In `C` this operator is expressed as `&`
- In this case it checks it against the value of `1`
  - Which means the output will be a 1, **only** if `value_X` **AND** `value_Y` are `1`!
  - A quick example:
    - Because the number 1 in binary could be represented as `001`, and the number 4 as `100`, the `AND` operation of this will **equal to 0**.
    - So: `if 0 == 0` then add 3, `else` subract 7
  

**If this is confusing for you, just know this:**
- All equal numbers will always end with a `0` in binary!
- So the `AND` operation using a `1` will always equal in `0` _if it's an even number_!
- <img width="623" height="647" alt="2026-02-22-03:31:31" src="https://github.com/user-attachments/assets/377eb070-c140-4d6b-8970-74a68cd03e64" /> 



-----

# C) Nora Crackme's 3 & 4
We already did a few of these [previously](<https://github.com/Ali-Mikael/Application-hacking/blob/main/h4-Disaddembly.md#d-nora>), but the 3 and 4 are new so let's get cracking!
Unfortunately my time ran out. It's 3am and the task has to be given in before 8AM. I don't feel like doing an all nighter, so good night i'll figure out the rest later! 🫡


### This is later:

I found the `check_pw` function using
```bash
$ objdump -d crackme03.64
```

```asm
0000000000001159 <check_pw>:
    1159:	b8 00 00 00 00       	mov    $0x0,%eax
    115e:	0f b6 0c 02          	movzbl (%rdx,%rax,1),%ecx
    1162:	02 0c 06             	add    (%rsi,%rax,1),%cl
    1165:	38 0c 07             	cmp    %cl,(%rdi,%rax,1)
    1168:	75 17                	jne    1181 <check_pw+0x28>
    116a:	80 7c 06 01 00       	cmpb   $0x0,0x1(%rsi,%rax,1)
    116f:	74 16                	je     1187 <check_pw+0x2e>
    1171:	48 83 c0 01          	add    $0x1,%rax
    1175:	80 3c 07 00          	cmpb   $0x0,(%rdi,%rax,1)
    1179:	75 e3                	jne    115e <check_pw+0x5>
    117b:	b8 01 00 00 00       	mov    $0x1,%eax
    1180:	c3                   	ret
    1181:	b8 00 00 00 00       	mov    $0x0,%eax
    1186:	c3                   	ret
    1187:	b8 01 00 00 00       	mov    $0x1,%eax
    118c:	c3                   	ret
```

