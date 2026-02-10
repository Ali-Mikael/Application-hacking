# The workhorse
<img width="1473" height="848" alt="Screenshot from 2026-02-10 16-23-21" src="https://github.com/user-attachments/assets/e41794df-ec24-48fa-9001-4127d797f876" />


# A) Lab0
Objective
- Investigate what's wrong with the [program](<https://terokarvinen.com/application-hacking/lab1.zip>) and how to fix it


# First things first
Unzip the target, remove zip file and move into the directory
```
$ unzip lab1.zip && rm lab1.zip && cd lab1
```

**We now have:**
- The src code
- The binary
- Build instructions
- <img width="847" height="91" alt="Screenshot from 2026-02-09 21-01-05" src="https://github.com/user-attachments/assets/c5e65d37-f95c-4bae-98ff-206cac07f146" />


> [!NOTE]
> While the original source code is still available, for the purpose of this exercise I will refrain from using it in order to find the problem
>
> When we do end up finding the bug, we might then fix it, compile again, and test if it works

We start off by doing a quick check what we're working with:
- <img width="1478" height="159" alt="Screenshot from 2026-02-10 18-15-22" src="https://github.com/user-attachments/assets/043321ef-217a-42cd-92d8-46f3f1a98f93" />

We run the program:
- <img width="1478" height="238" alt="Screenshot from 2026-02-10 18-16-48" src="https://github.com/user-attachments/assets/494947d1-dee6-4214-9231-3f1b06e31ad3" />

None the wiser, we move on.


# Debugging
We start the Gnu Debugger by issuing the command:
```
$ gdb gdb_example1
```
We then get the `(gdb)` prompt. And if we want to see the C code being executed side by side, we can issue the command:
```
$ (gdb) tui enable
```
(TUI = Text User Interface. which you can also open at the same time as the program if you start GDB with the `--tui` parameter)



Which will result in:
- <img width="1476" height="1077" alt="Screenshot from 2026-02-10 18-08-29" src="https://github.com/user-attachments/assets/a3c20ab6-7051-454d-af35-5b1c9849e07d" />
- We can now navigate the code line by line using the arrow keys


On the prompt, we issue the command:
```
(gdb) break 5
```
- This will effectively set the breakpoint at the first line inside the `print_scrambled()` function

By typing:
```
(gdb) run
```

We run the program and stop execution at the break point.

We also want to keep track of the variables, so we issue two more commands:
```
(gdb) display message
(gdb) display i
```
This will show us the values after each step we take.

> [!TIP]
>
> Execution control commands
> - `continue`: _Resume execution until next breakpoint_
> - `next`: _Execute next line, step over functions_
> - `step`: _Execute next line, step into functions_
> - `finish`: _Run until current function returns_
> - `until`: _Continue until a certain line or loop ends_

Now that the program is runnning, and we want to move forward a line, we use the `step` command (or abbreviated: `s`)




