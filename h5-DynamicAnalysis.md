# The workhorse
<img width="1473" height="848" alt="Screenshot from 2026-02-10 16-23-21" src="https://github.com/user-attachments/assets/e41794df-ec24-48fa-9001-4127d797f876" />


# A) Lab1
**Objective**
- Investigate what's wrong with the [program](<https://terokarvinen.com/application-hacking/lab1.zip>) and how to fix it


## First things first
Unzip the target, remove zip file and move into the directory
```bash
$ unzip lab1.zip && rm lab1.zip && cd lab1
```

**We now have:**
- The src code
- The binary
- Build instructions
- <img width="847" height="91" alt="Screenshot from 2026-02-09 21-01-05" src="https://github.com/user-attachments/assets/c5e65d37-f95c-4bae-98ff-206cac07f146" />


We start off by doing a quick sanity check:
- <img width="1478" height="159" alt="Screenshot from 2026-02-10 18-15-22" src="https://github.com/user-attachments/assets/043321ef-217a-42cd-92d8-46f3f1a98f93" />

Run the program and see what happens:
- <img width="1478" height="238" alt="Screenshot from 2026-02-10 18-16-48" src="https://github.com/user-attachments/assets/494947d1-dee6-4214-9231-3f1b06e31ad3" />

We learn that the program is causing a segmentation fault. So we investigate ->


## Debugging
We start the _GNU Debugger_ by issuing the command:
```bash
$ gdb gdb_example1
```
We then get the `(gdb)` prompt. 

If we want to see the `C` code being executed side by side, we can issue the command:
```bash
$ (gdb) tui enable
```
- TUI = Text User Interface
- You can also skip this step by initially starting GDB with the `-tui` parameter


On the prompt, we issue commands:
```bash
(gdb) break 5 
(gdb) break 14 
```
- This will effectively set breakpoints at the **first line** inside the `print_scrambled()` and `main()` **functions**
- We can also do the abbreviated version: `b <line>`

What it looks like:
- <img width="1478" height="708" alt="Screenshot from 2026-02-11 11-43-27" src="https://github.com/user-attachments/assets/ef9590e0-8e5a-4e92-b347-e9384da39b37" />
- <img width="1533" height="826" alt="Screenshot from 2026-02-11 11-45-53" src="https://github.com/user-attachments/assets/724dd9f6-65c7-43cb-b580-f7a2305048b7" />


We run the program and stop execution at the first break point inside `main()`

```bash
(gdb) run
```
- <img width="931" height="179" alt="Screenshot from 2026-02-12 08-25-31" src="https://github.com/user-attachments/assets/eee898ef-e0c5-44b8-b603-ac84fc8ba199" />


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
> **Note also** that you can abbreviate the commands as long as there is no ambiguity
>
> Meaning if you have two commands: `continue` and `complete`, you have to type `con` or `com`


```bash
(gdb) i lo
```
= **info locals**
- We get the values for the local variables
- <img width="976" height="128" alt="Screenshot from 2026-02-12 08-25-16" src="https://github.com/user-attachments/assets/28aeb5f9-93fd-4da7-931a-28193ba1f6a9" />


We then move forward _one_ line and inspect the variables again with `i lo`
- We can see that `bad_message` is null = `0x0`
- <img width="721" height="130" alt="Screenshot from 2026-02-12 08-30-43" src="https://github.com/user-attachments/assets/193fdd84-1013-468c-8361-2162be8f4054" />


Once we get to `line 17` where a function call is made, we now use the command `s` (step)
- We get an informational message of what's happening:
  - <img width="1648" height="125" alt="Screenshot from 2026-02-12 08-45-47" src="https://github.com/user-attachments/assets/8658b0d8-00db-4ebc-a2ba-32bdf4597fad" />
  - <img width="851" height="363" alt="Screenshot from 2026-02-12 08-45-32" src="https://github.com/user-attachments/assets/68ca5aa5-6132-451a-90d8-b782cbba05d9" />


Instead of manually typing `print` or `i lo`, we want to keep track of the variables automatically, so we issue the command:
```
(gdb) display message
```
This will show us the value after each step we take
- <img width="744" height="399" alt="Screenshot from 2026-02-12 08-58-06" src="https://github.com/user-attachments/assets/4b1276ea-283a-4674-a728-f0e034b53991" />

We skip to the end of the loop with the `until` command:
- <img width="564" height="202" alt="Screenshot from 2026-02-12 09-00-55" src="https://github.com/user-attachments/assets/2f9d0aa3-d31c-4847-8bdf-e42c9596fff6" />
- We're able to see the scrambled message

But what happens when we try to scramble a variable with the value `null`?
- It first passes the null variable to the function as an argument
  - `print_scrambled(bad_message)`
  - <img width="1179" height="163" alt="Screenshot from 2026-02-12 09-01-29" src="https://github.com/user-attachments/assets/e3500f1e-3f13-4678-8cfd-bd61099380a2" />
- And what follows is a segmentation fault:
  - <img width="1310" height="171" alt="Screenshot from 2026-02-12 09-02-04" src="https://github.com/user-attachments/assets/8a4e42d5-8b58-4b61-b9be-b3821f30595d" />

## What is a segmentation fault?
Simply put:
- The program tries to access a memory location without having the permission to do so
- The hardware will sense this and execute a jump to the OS
- ON UNIX family platforms the OS will normally announce that the program has caused a seg fault and stop the execution of the program

(Norman M. Peter Jay S, 2008)

## Fixing it
The problem:
- The program is trying to access a memory location it has no **access rights** to, which is the `0x0`
- Why? Because the variable is `null`


We fix it by
- Changing the value to **not null** (i'm doing it straight from the source code, as we still have access to it)
- <img width="646" height="132" alt="Screenshot from 2026-02-12 11-58-34" src="https://github.com/user-attachments/assets/71a7e0d1-d04c-483b-bdf6-7841ad164e42" />

We then recocompile:
```
$ gcc -g -o example1 gdb_example1.c
```
By running the recompiled program `example1` alongside the faulty one, we can see that we got rid off the seg fault:
- <img width="1091" height="242" alt="Screenshot from 2026-02-12 12-02-02" src="https://github.com/user-attachments/assets/2cb3afb3-a7c3-4634-a574-8d6c5f0f052c" />



**Help received**
- Norman M. Peter Jay S (2008) [The Art of Debugging With GDB, DDD and Eclipse](<https://zhjwpku.com/assets/pdf/books/The.Art.of.Debugging.with.GDB.DDD.and.Eclipse.pdf>)
- [The GDB developer's GNU Debugger tutorial, Part 1: Getting started with the debugger](<https://developers.redhat.com/articles/the-gdb-developers-gnu-debugger-tutorial-part-1-getting-started-with-the-debugger#>)
- [GDB command](<https://www.tutorialspoint.com/gnu_debugger/gdb_commands.htm>)
- [Quick Guide to gdb: The GNU Debugger](<https://kauffman77.github.io/tutorials/gdb.html>)








