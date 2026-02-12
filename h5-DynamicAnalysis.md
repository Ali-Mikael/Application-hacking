# The workhorse
<img width="1473" height="848" alt="Screenshot from 2026-02-10 16-23-21" src="https://github.com/user-attachments/assets/e41794df-ec24-48fa-9001-4127d797f876" />


# A) Lab0
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


> [!NOTE]
> While the original source code is still available, for the purpose of this exercise I will refrain from using it in order to find the problem
>
> When we do end up finding the bug we might then fix it, compile again, and test if it works


We start off by doing a quick sanity check:
- <img width="1478" height="159" alt="Screenshot from 2026-02-10 18-15-22" src="https://github.com/user-attachments/assets/043321ef-217a-42cd-92d8-46f3f1a98f93" />

Run the program and see what happens:
- <img width="1478" height="238" alt="Screenshot from 2026-02-10 18-16-48" src="https://github.com/user-attachments/assets/494947d1-dee6-4214-9231-3f1b06e31ad3" />

None the wiser, we move on!


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
- You can also skip this step by starting GDB with the `-tui` parameter


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
> - `next n`: _Execute N lines of code_
> - `step`: _Execute next line, **step into** functions_
> - `step n`: _Execute N lines of code, step into functions_
> - `continue`: _Resume execution until next breakpoint_
> - `finish`: _Run until current function returns_
> - `until`: _Continue until a certain line or loop ends_


```
(gdb) i lo
```
Meaning = `info locals`, we get information about the local variables
- <img width="976" height="128" alt="Screenshot from 2026-02-12 08-25-16" src="https://github.com/user-attachments/assets/28aeb5f9-93fd-4da7-931a-28193ba1f6a9" />


We then move forward one line by typing `n` meaning `next` and inspect the variables again with `i lo`
- We can see that `bad_message` is null = `0x0`
- <img width="721" height="130" alt="Screenshot from 2026-02-12 08-30-43" src="https://github.com/user-attachments/assets/193fdd84-1013-468c-8361-2162be8f4054" />

Once we get to `line 17` where a function call is made, we now use the command `s` meaning `step`
- We get an informational message of what's happening:
  - <img width="1648" height="125" alt="Screenshot from 2026-02-12 08-45-47" src="https://github.com/user-attachments/assets/8658b0d8-00db-4ebc-a2ba-32bdf4597fad" />
  - <img width="851" height="363" alt="Screenshot from 2026-02-12 08-45-32" src="https://github.com/user-attachments/assets/68ca5aa5-6132-451a-90d8-b782cbba05d9" />


Instead of manually typing `print` or `i lo`, we want to keep track of the variables automatically, so we issue the command:
```
(gdb) display message
```
This will show us the value after each step we take (`next`)
- <img width="744" height="399" alt="Screenshot from 2026-02-12 08-58-06" src="https://github.com/user-attachments/assets/4b1276ea-283a-4674-a728-f0e034b53991" />






