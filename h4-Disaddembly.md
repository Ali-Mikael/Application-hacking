# X) Read/Watch/Listen and Summarize
Hammond 2022 - [GHIDRA for Reverse Engineering](<https://www.youtube.com/watch?v=oTD_ki86c9I>) (video)
- Before pulling out ghidra, it might prove itself fruitful to poke the ice with tools such as `ltrace`, `strace` and `objdump`
- Once the file is imported and analyzed in ghidra, first order of business is to locate strings
- Find the main function, familiarise yourself with the code and rename variables if applicable/possible/feasible
- Pay attention to the way the function behaves
- The solution for the example in the video was to find a `hex string` and pass the decimal representation of it into the program in order to capture the flag

-----

# Infrastructure for completing objectives
**HOST a.k.a The Provider**
- **Lenovo ThinkPad L490**
  - _CPU:_ Intel i3-8145U (4) @ 3.900GHz 
  - _GPU:_ Intel WhiskeyLake-U GT2 UHD Graphics 620
  - _Memory:_ 8GB
  - _Disk size:_ 256GB
  - _OS:_ Ubuntu 24.04.3 LTS x86_64
  - _Kernel:_ 6.8.0-90-generic
- _Virtualization:_ KVM/QEMU Standard PC (Q35 + ICH9, 2009) (pc-q35-8.2)
- _Managed by:_ Virtual Machine Manager v.4.1.0, powered by libvirt.

**VM a.k.a The Playmaker a.k.a The Attacker**
- _OS:_ Kali GNU/Linux Rolling x86_64
- _Kernel:_ Linux 6.18.5+kali-amd64
- _Memory:_ 4GB
- _Storage:_ 28GB


-----

# A) Install Ghidra
For `Kali Linux`:
```bash
$ sudo apt update && sudo apt install -y ghidra
```
Run it:
```
$ ghidra
```
<img width="1202" height="738" alt="Screenshot from 2026-01-27 21-54-40" src="https://github.com/user-attachments/assets/38020368-4485-4cdc-b92a-e287703f46c4" />


------


# B) Rever-C
**OBJECTIVE**
- Reverse engineer the `packd` binary to C language using `ghidra`
- Find the main program
- Give variables descriptive names
- Explain the programs operation
- Solve the task from the binary, without the original source code.

Target can be installed from [here!](<https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip>)


## Reverse engineering
I created a new directory for storing ghidra projects, and within it a directory for the current task.
```bash
$ mkdir -p ghidraProjects/packd/
```
Fire up `ghidra` and import the binary:
- <img width="668" height="525" alt="Screenshot from 2026-02-04 23-00-23" src="https://github.com/user-attachments/assets/c6f4e5f5-21e1-4832-9267-7c35c2257808" />
- <img width="688" height="373" alt="Screenshot from 2026-02-04 23-02-01" src="https://github.com/user-attachments/assets/07133052-f5b5-4122-a9f1-1bb1c9f10537" />


> [!NOTE]
> It should be mentioned that i'm using the **same exact copy** of the binary in the **h3-task**, so it's already **unpacked**!
>
> If this sounds unfamiliar, check [this out](<https://github.com/Ali-Mikael/Application-hacking/blob/main/h3-NoStringsAttached.md#c-packd>)


#### We then let ghidra analyze the file and produce high level C code and assembly instructions:
<img width="1317" height="790" alt="Screenshot from 2026-02-04 23-04-21" src="https://github.com/user-attachments/assets/d9c23ccd-ded2-44e0-b65b-481f6139cbbc" />

> [!NOTE]
> **Regarding the decompiled C code**
>
> By analysing the binary Ghidra generates what is known as `P-Code`, which is the _intermediary representation_ of the _instruction set_
>
> "_Fundamentally, p-code works by translating individual processor instructions into a sequence of p-code operations that take parts of the processor state as input and output variables (varnodes)_" - P-Code Reference manual
> 
> The _high level C code_ you then see in the `decompiler` is actually `C pseudocode` derived from the aforementioned p-code
>
> Sources:
>- [P-Code Reference manual](<https://ghidra.re/ghidra_docs/languages/html/pcoderef.html>)
>- [HackadayU: Reverse Engineering with Ghidra Class 1 (starting at 52:29)](<https://www.youtube.com/watch?v=d4Pgi5XML8E&list=PL_tws4AXg7auglkFo6ZRoWGXnWL0FHAEi&index=2>)

#### We're able to navigate through different parts of the code using the symbol tree on the left:
<img width="1310" height="773" alt="Screenshot from 2026-02-04 23-08-01" src="https://github.com/user-attachments/assets/845e87d9-3ef8-4f37-af75-b8c1e574e2f3" />


#### Lucky for us there's not that many functions and it's a small program. Plus the main program is called main, so we find it pretty easily:
<img width="831" height="421" alt="Screenshot from 2026-02-04 23-12-10" src="https://github.com/user-attachments/assets/bc2335fc-0743-4e17-bae8-9aa86072c5ad" />


#### We then rename the variables to something comprehensible, so that it's easier to read and understand the function:
Highlight the variable you want to change and press `L`
- <img width="723" height="252" alt="Screenshot from 2026-02-04 23-14-17" src="https://github.com/user-attachments/assets/b972d6a7-4d31-4405-be35-84288d7124e8" />
- <img width="766" height="285" alt="Screenshot from 2026-02-04 23-28-29" src="https://github.com/user-attachments/assets/7daca284-0abc-4fc5-864f-e8afc313f3f2" />

#### How does the program work?
- It _initialises a password variable_ and later on in the function populates that variable with a number
- The number in question is the **result** of `comparing` the **password** with **user input**
- If the strings `match` == 0
  - 0 means the comparison operation was successful and the strings are equal
- This is all done using a simple `if else` statement
  - `if true` == the flag is provided
  - `else` (anything else than 0) == no bonus :/

------


# C) Backwards
**OBJECTIVE**
- Modify the `passtr` programs's binary so that it **accepts all** passwords **except** the correct one
  - Without the original source code of course!
- Demonstrate with tests that your solution works

## Manipulation
We import the second program called `passtr` 
- Analyze it
- Open up the main function
- Give variables descriptive names
- <img width="833" height="447" alt="Screenshot from 2026-02-04 23-37-18" src="https://github.com/user-attachments/assets/ac8830d5-3d42-4e6f-a169-ae6f74be359f" />



**I opened up the assembly instructions alongside the decompiler and highlighed the function I wanted to change so that the _corresponding instructions_ would be highlighted as well:**
- <img width="1314" height="539" alt="Screenshot from 2026-02-05 18-45-09" src="https://github.com/user-attachments/assets/25426108-b8dd-4c30-a9e5-733dcaf1d808" />

### I knew I had to modify the assembly instructions somehow
- So I did some googling on instructions, syntax and how `if else` logic is built in assembly
- I realised that after _comparing the passwords_ there's an instruction following it == `JNZ`, which will **jump** to the **"sorry no bonus"** if value is **not zero**
- `JNZ` = Jump Not Zero
- And after the instructions `JNZ` the target of the operation is specified == `LAB_001011b6`
  - Which signifies **some code to be executed** at a **specified address** (Chris Eagle, 2020)
  - `LAB` = some code
  - `001011b6` = the address
- Otherwise it will give the flag, which is next up in the execution order if we don't jump
- <img width="972" height="247" alt="Screenshot from 2026-02-05 18-55-32" src="https://github.com/user-attachments/assets/83337733-a322-4375-a15f-46f2bbb1c211" />


### This is what I came up with:

Just remove the `N` innit 😂
- `Ctrl+Shift+G` to `Patch Instruction` (modify)
- Remove the `N` and apply
- <img width="813" height="96" alt="Screenshot from 2026-02-05 18-59-44" src="https://github.com/user-attachments/assets/27a6cdb5-e53b-4e8e-b196-b818d31ecfa9" />
- Save the file `Ctrl+S`
- Export it back to binary
  - Ghidra takes care of recompiling!
- Then make the binary executable again
- <img width="1670" height="398" alt="Screenshot from 2026-02-05 19-07-02" src="https://github.com/user-attachments/assets/4736f74b-e063-4ed7-bdbc-358949b18469" />


### We then test that it works by typing in 2 wrong passwords followed by the correct one:
<img width="939" height="509" alt="Screenshot from 2026-02-05 19-07-50" src="https://github.com/user-attachments/assets/c34743cb-22ca-4864-bcf7-3d68d4e45e23" />


**Explanation:**
- We reverse the logic 
- Earlier the program made sure that the `password` and `user_input` strings were equivailent before providing the flag
- After modification: only strings that are NOT equal to the password will get the flag!
- In high level code this means we switch the `==` to `!=`


**Help used:**
- [An article on modifying a port number from binary](<https://blog.cjearls.io/2019/04/editing-executable-binary-file-with.html>)
- [Assembly instructions](<https://www.tutorialspoint.com/assembly_programming/assembly_conditions.htm>)
- [The Ghidra Book by Chris Eagle 2020, Chapter 7](<https://learning.oreilly.com/library/view/the-ghidra-book/9781098125684/>)

-----


# D) Nora
**OBJECTIVE:**
- Compile the [NoraCodes/crackmes](<https://github.com/NoraCodes/crackmes>) to binary


Download the repository containing the targets
```
$ git clone https://github.com/NoraCodes/crackmes.git
```

There's about 20 `C`-files in the repo and we have to compile them all to binary. 

I didn't feel like doing any manual labour so i created a quick `for loop`
```bash
for crack in *.c; do
  gcc "$crack" -o "${crack%.c}"
done
```
<img width="1304" height="474" alt="Screenshot from 2026-02-06 09-33-47" src="https://github.com/user-attachments/assets/34c82f59-2d18-4815-b042-dc6aa1e4e672" />

Now we're ready to get cracking!

-----

# E) crackme01
**OBJECTIVE**
- Solve the binary

## Solving it
First things first let's execute the binary and see what happens
- <img width="488" height="197" alt="Screenshot from 2026-02-06 09-38-39" src="https://github.com/user-attachments/assets/dc22c016-b22c-4e3c-b5a9-e9f2ce1a0472" />

Because the original source code is off limits and the machine code is alien language, I decided to exract some human readable content from the executable by using our old friend `strings`. 

#### And wouldn't you know, it delivered once again: 
<img width="457" height="541" alt="Screenshot from 2026-02-06 09-39-29" src="https://github.com/user-attachments/assets/9e0f6663-1e84-4f4a-a78b-83a3c5569b48" />


<img width="468" height="121" alt="Screenshot from 2026-02-06 09-46-25" src="https://github.com/user-attachments/assets/f2bdba7b-1685-411e-a05d-84bfdc23088a" />

-----


# E.2) crackme01e
**OBJECTIVE**
- Same as above

## Solving it
Same kind of workflow
- <img width="650" height="710" alt="Screenshot from 2026-02-06 09-50-31" src="https://github.com/user-attachments/assets/a37917f5-dae2-47f9-be33-608aebfb9656" />


This time we had to _escape a special character_ in order to complete the objective 
- Either using a forward slash
- or
- Single quotes: so the shell interprets it _literally_ and doesn't expand it
- <img width="512" height="218" alt="Screenshot from 2026-02-07 12-04-38" src="https://github.com/user-attachments/assets/c695e774-6bde-411b-966d-3bad39f01a25" />


-----


# F) crackme02
**OBJECTIVE**
- Locate and name the main program's variables from the reverse-engineered binary
- Explain the program's operation
- Solve the binary

## Reverse Engineering, Explaining and Renaming

Naturally we start off by pulling out big gun a.k.a `ghidra`
- We created a new project and imported the `crackme02` binary
- Ghidra then did the heavy lifting and analysed the file for us:
  - Producing `assembly instructions` and `C code`

The `main` function:
```C
undefined8 main(int param_1,long param_2)

{
  undefined8 uVar1;
  int local_c;
  
  if (param_1 == 2) {
    for (local_c = 0;
        ("password1"[local_c] != '\0' && (*(char *)((long)local_c + *(long *)(param_2 + 8)) != '\0')
        ); local_c = local_c + 1) {
      if ("password1"[local_c] + -1 != (int)*(char *)((long)local_c + *(long *)(param_2 + 8))) {
        printf("No, %s is not correct.\n",*(undefined8 *)(param_2 + 8));
        return 1;
      }
    }
    printf("Yes, %s is correct!\n",*(undefined8 *)(param_2 + 8));
    uVar1 = 0;
  }
  else {
    puts("Need exactly one argument.");
    uVar1 = 0xffffffff;
  }
  return uVar1;
}
```


I figured the first variable `uVar1` inside the function might be used as the `exit code`, as the function will always return with it no matter what. 

**To support our hypothesis**
- We can see that it's used in the `if else` clause:
  - If the password is **correct** the variable is set to `0` and later returned
    - Indicating that everything went well.
  - If the password is **wrong** the variable is set to `0xffffffff`
    - Which will result in returning the highest possible exit status (which is 255 in Linux)


**We found 4 variables to rename:**
- `cliArgs` 
- `userInput`
- `exitCode`
- `i` (the index used in the for loop).

#### This makes it easier to read and understand the program
<img width="969" height="540" alt="Screenshot from 2026-02-06 16-25-22" src="https://github.com/user-attachments/assets/f3370ee2-f020-4627-9e25-a48913cb11fa" />

## So how does it work?

**The main function takes 2 parameters**
1. _Amount of CLI arguments_
  - For clarity: calling the program itself (`$ ./<program>`) counts as an argument on the CLI!
2. _Parameter used when calling the program_

**What happens next?**
- If the _number of command line arguments_ are **NOT** == 2 (`program invocation` + `parameter`) the program will exit
- If the arguments are **exactly 2** in number, the function will then do some operation on the password and user input


#### This is achieved in the following manner:

> [!NOTE]
> This is pseudocode derived from the representation of the original source code to drive the point across!
>

```C
if (cliArgs == 2){
  -- doing some magic on strings --
}
else {
  puts("Exactly one argument needed");
  exitCode = FAIL;
}
```


### The magic

The following section in the code really caught my interest:
```C
if ("password1"[i] + -1
```
- It seems like the program is taking each character of the password and shifting it back by one
  - a.k.a the `ceasar cipher`
- It also seemed as the loop is not accounting for the length of the input
  - Mainly because the for loop ends when **either** string ends
- Which takes us to the next step:


The letter before `p` is `o` so let's give it a go 
> It can't be wrong if it rhymes...

<img width="524" height="107" alt="Screenshot from 2026-02-06 16-45-27" src="https://github.com/user-attachments/assets/3e293b28-560f-4ceb-b655-ff1fe850d619" />


And wouldn't you know, didn't even have to type in the whole `password` == ``o`rrvnqc0``


Here's an [ascii table](<https://www.ascii-code.com/>) for reference. 

For the most part just knowing the alphabet was enough here. I just needed to know what comes before `a`, which was the backtick.

-----


# I) Optional: solve the crackme02e
**You already know the drill!**

<img width="512" height="523" alt="Screenshot from 2026-02-06 22-03-39" src="https://github.com/user-attachments/assets/58e0fb00-2f28-4dbd-aa5d-13886d466691" />


<img width="513" height="105" alt="Screenshot from 2026-02-06 22-04-11" src="https://github.com/user-attachments/assets/7329c63d-bc33-4929-a4fe-ad40b24adb90" />


This is the main function ghidra spit out for us (I already took the liberty to rename a few entry level variables):
```C
undefined8 main(int cliArgs,long userInput)

{
  undefined8 exitCode;
  int i;
  
  if (cliArgs == 2) {
    for (i = 0; ((&DAT_0010201f)[i] != '\0' &&
                (*(char *)((long)i + *(long *)(userInput + 8)) != '\0')); i = i + 1) {
      if ((char)(&DAT_0010201f)[i] + -2 != (int)*(char *)((long)i + *(long *)(userInput + 8))) {
        printf("No, %s is not correct.\n",*(undefined8 *)(userInput + 8));
        return 1;
      }
    }
    printf("Yes, %s is correct!\n",*(undefined8 *)(userInput + 8));
    exitCode = 0;
  }
  else {
    puts("Need exactly one argument.");
    exitCode = 0xffffffff;
  }
  return exitCode;
```

My attention immediately went to the `&DAT_0010201f` variable, which seems to be be the password
- This time around the password is not in clear text though
- It's rather a **pointer** to **some data** located at a **specified address**
  - The `DAT` prefix signifies `data`
  - `0010201f` is the address
  - The Ghidra Book Chapter 7 did a great job explaining this concept!


I created an array from it using ghidra:
- Double click the variable which takes us to the `assembly view`
- Then right click -> Data -> `char`
- Right click **again** -> Data -> `Create Array`, where I chose 8 as the length
- <img width="1265" height="420" alt="Screenshot from 2026-02-07 14-31-01" src="https://github.com/user-attachments/assets/e14b7ac8-e3d1-4b35-ac83-3ff0d3f509e5" />


As a result, we have the password string as a char array and we can now spot it in the decompiler
- <img width="1008" height="442" alt="Screenshot from 2026-02-07 14-34-55" src="https://github.com/user-attachments/assets/4f24cffa-a79c-470c-9f0c-e0f842a3d2a2" />
- <img width="909" height="184" alt="Screenshot from 2026-02-07 14-34-17" src="https://github.com/user-attachments/assets/315bb68b-2efa-4078-abf0-ad217559b7c9" />


**Further clean up**
- I also changed the **function return type** to `int` and **userInput type** to `char **`
  - `function` we modify by right clicking it -> `Edit Function Signature` -> `builtin` -> `int`
  - `userInput parameter` we modify by placing the cursor on top of it, then -> `Ctrl+L` (Retype Variable) -> `char **`
    - By specifying the correct type, the code then becomes more readable and easier to interpret


#### DevOps who? Call me the CleanOps engineer!
```C
int main(int cliArgs,char **userInput)

{
  int exitCode;
  int i;
  
  if (cliArgs == 2) {
    for (i = 0; ("yuvmnpoi"[i] != '\0' && (userInput[1][i] != '\0')); i = i + 1) {
      if ("yuvmnpoi"[i] + -2 != (int)userInput[1][i]) {
        printf("No, %s is not correct.\n",userInput[1]);
        return 1;
      }
    }
    printf("Yes, %s is correct!\n",userInput[1]);
    exitCode = 0;
  }
  else {
    puts("Need exactly one argument.");
    exitCode = -1;
  }
  return exitCode;

```

Now that we have the code, we quickly realise it does the same thing as previously, but shifting with a 2 instead of 1 this time.

Let's put that to test:
- `yuvmnpoi` shifted back by 2 ascii is == `wstklnmg`
- <img width="617" height="395" alt="Screenshot from 2026-02-07 16-06-52" src="https://github.com/user-attachments/assets/2a8018bd-3716-41ea-b0a4-9e4d81998e6f" />


Again, the loop ends when either string runs out, so we can type either _one character_ of the string, the whole string or anything in between
- Fu**it, we can even go _above and beyond_ as the password ends and doesn't have any more characters to compare
  - <img width="635" height="131" alt="Screenshot from 2026-02-07 16-09-55" src="https://github.com/user-attachments/assets/71bbdc4c-06dc-4e89-bb5f-9f7d864237ec" />
- But as soon as the first character is different, we're denied entry:
  - <img width="635" height="131" alt="Screenshot from 2026-02-07 16-10-12" src="https://github.com/user-attachments/assets/51c3969e-1883-4dc5-a06e-079741b4e649" />


**Help received**
- From this [video](<https://www.youtube.com/watch?v=uyWVztMHWtk&list=PL_tws4AXg7auglkFo6ZRoWGXnWL0FHAEi&index=3>) by Matthew Alt produced by Hackaday (The whole video was interesting, but I specifically learned how to create the char array from the section starting at the **53:12** mark)
- As well as [The Ghidra Book](<https://learning.oreilly.com/library/view/the-ghidra-book/9781098125684/>) by Chris Eagle 2020
  - Chapter 7: Disassembly Manipulation
  - Chapter 19: The Ghidra Decompiler

