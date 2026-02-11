# The workhorse
<img width="1473" height="848" alt="Screenshot from 2026-02-10 16-23-21" src="https://github.com/user-attachments/assets/e41794df-ec24-48fa-9001-4127d797f876" />


# A) Lab0
Objective
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


> [!NOTE]
> While the original source code is still available, for the purpose of this exercise I will refrain from using it in order to find the problem
>
> When we do end up finding the bug we might then fix it, compile again, and test if it works


We start off by doing a quick check:
- <img width="1478" height="159" alt="Screenshot from 2026-02-10 18-15-22" src="https://github.com/user-attachments/assets/043321ef-217a-42cd-92d8-46f3f1a98f93" />

Run the program and see what happens:
- <img width="1478" height="238" alt="Screenshot from 2026-02-10 18-16-48" src="https://github.com/user-attachments/assets/494947d1-dee6-4214-9231-3f1b06e31ad3" />

None the wiser, we move on!


## Debugging
We start the Gnu Debugger by issuing the command:
```bash
$ gdb gdb_example1
```
We then get the `(gdb)` prompt. 

If we want to see the `C` code being executed side by side, we can issue the command:
```bash
$ (gdb) tui enable
```
- TUI = Text User Interface
- You can also skip this step by starting GDB with the `--tui` parameter


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


By typing:
```
(gdb) run
```
We run the program and stop execution at the first break point.


We also want to keep track of the variables, so we issue two more commands:
```
(gdb) display message
(gdb) display i
```
This will show us the values after each step we take.


> [!TIP]
>
> **Execution control commands**
> - `continue`: _Resume execution until next breakpoint_
> - `next`: _Execute next line, **step over** functions_
> - `step`: _Execute next line, **step into** functions_
> - `finish`: _Run until current function returns_
> - `until`: _Continue until a certain line or loop ends_


Now that the program is runnning, and we want to move forward a line, we use the `step` command (or abbreviated: `s`)




