# X) Read/Watch/Listen and Summarize
Hammond 2022 - [GHIDRA for Reverse Engineering](<https://www.youtube.com/watch?v=oTD_ki86c9I>) (video)
- 

# A) Install Ghidra
For `kali linux`:
```
$ sudo apt update && sudo apt install -y ghidra
```
Run it by typing `$ ghidra`:

<img width="1202" height="738" alt="Screenshot from 2026-01-27 21-54-40" src="https://github.com/user-attachments/assets/38020368-4485-4cdc-b92a-e287703f46c4" />


------


# B) Rever-C
**OBJECTIVE**
- Reverse engineer the packd binary to C language using `ghidra`
- Find the main program
- Give variables descriptive names
- Explain the programs operation
- Solve the task from the binary, without the original source code.

Target can be installed from [here!](<https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip>)


## Reverse engineering
I created a new directory for my project
```bash
$ mkdir -p ghidraProjects/packd/
```
Then I fired up `ghidra` and opened up the binary. 

<img width="668" height="525" alt="Screenshot from 2026-02-04 23-00-23" src="https://github.com/user-attachments/assets/c6f4e5f5-21e1-4832-9267-7c35c2257808" />

<img width="688" height="373" alt="Screenshot from 2026-02-04 23-02-01" src="https://github.com/user-attachments/assets/07133052-f5b5-4122-a9f1-1bb1c9f10537" />


> [!NOTE]
>
> It should be mentioned that i'm using the _same exact copy_ of the binary in the `h3 task`, so it's already _unpacked_.
>
> If this sounds unfamiliar, check [this out](<https://github.com/Ali-Mikael/Application-hacking/blob/main/h3-NoStringsAttached.md#c-packd>)


#### We then let ghidra do the heavy lifting for us, and analyze the contents in order to reverse engineer it to C:
<img width="1317" height="790" alt="Screenshot from 2026-02-04 23-04-21" src="https://github.com/user-attachments/assets/d9c23ccd-ded2-44e0-b65b-481f6139cbbc" />


#### We're able to navigate through different parts of the code using the symbol tree:
<img width="1310" height="773" alt="Screenshot from 2026-02-04 23-08-01" src="https://github.com/user-attachments/assets/845e87d9-3ef8-4f37-af75-b8c1e574e2f3" />


#### Luckily for us, there's not that many functions and it's a pretty small program. Plus the main program is called main, so we find it pretty easy:
<img width="831" height="421" alt="Screenshot from 2026-02-04 23-12-10" src="https://github.com/user-attachments/assets/bc2335fc-0743-4e17-bae8-9aa86072c5ad" />


#### We then change the variable name to something more easily recognizable:
<img width="723" height="252" alt="Screenshot from 2026-02-04 23-14-17" src="https://github.com/user-attachments/assets/b972d6a7-4d31-4405-be35-84288d7124e8" />


#### We do the same for the user input. And like so, it's much easier to read the program now:
<img width="766" height="285" alt="Screenshot from 2026-02-04 23-28-29" src="https://github.com/user-attachments/assets/7daca284-0abc-4fc5-864f-e8afc313f3f2" />

### How does the program work?
The program is pretty simple. It initialises a variable with the password and then asks for user input.

The user input is then matched against the variable using a simple `if else` statement
- `if true` == the flag is provided
- `else` == no bonus :/

------


# C) Backwards
**OBJECTIVE**
- Modify the passtr programs's binary so that it accepts all passwords except the correct one
  - Without the original source code of course!
- Demonstrate with tests that your solution works

# Manipulation
We import the second program called `passtr` 

--> analyze it 

--> open up the main function

--> give variables descriptive names:

<img width="1039" height="356" alt="Screenshot from 2026-02-04 23-34-26" src="https://github.com/user-attachments/assets/f12f92aa-f8fe-426c-9a35-91d15ae60d8f" />

<img width="833" height="447" alt="Screenshot from 2026-02-04 23-37-18" src="https://github.com/user-attachments/assets/ac8830d5-3d42-4e6f-a169-ae6f74be359f" />


I opened up the assembly instructions alongside the decompiler and highlighed the function I wanted to change so that I would notice it in the instructions (also being automatically highlighted as a result):

<img width="1314" height="539" alt="Screenshot from 2026-02-05 18-45-09" src="https://github.com/user-attachments/assets/25426108-b8dd-4c30-a9e5-733dcaf1d808" />

I knew I had to modify the assembly somehow, so I did some googling on assembly syntax and how `if else` logic is built in assembly.

I realised that after _comparing the passwords_ there's an instruction after it == `JNZ`, which will jump to the **"sorry no bonus"** if value is **not zero**
- `JNZ` = Jump if Not Zero
- Otherwise it will give the flag, which is next up in the execution order if we don't jump.

<img width="972" height="247" alt="Screenshot from 2026-02-05 18-55-32" src="https://github.com/user-attachments/assets/83337733-a322-4375-a15f-46f2bbb1c211" />


### This is what I came up with:
Just remove the `N` innit. 😂

I pressed Ctrl+Shift+G to `Patch Instruction` (modify).

<img width="813" height="96" alt="Screenshot from 2026-02-05 18-59-44" src="https://github.com/user-attachments/assets/27a6cdb5-e53b-4e8e-b196-b818d31ecfa9" />


I then saved the file and _exported it_ to the same format as the original program (binary). Ghidra takes care of the recompilation! 

We also needed to make the binary executable after exporting:

<img width="1670" height="398" alt="Screenshot from 2026-02-05 19-07-02" src="https://github.com/user-attachments/assets/4736f74b-e063-4ed7-bdbc-358949b18469" />


### We then test it by typing in 2 wrong passwords followed by the correct one:
<img width="939" height="509" alt="Screenshot from 2026-02-05 19-07-50" src="https://github.com/user-attachments/assets/c34743cb-22ca-4864-bcf7-3d68d4e45e23" />


**Explanation:**
- We reverse the logic 
- Earlier the program made sure that the `password` and `user_input` strings were equivailent before providing the flag
- After modification: only strings that are NOT equal to the password will get the flag!
- In high level code this means we switch the `==` to `!=`


**Help used:**
- [An article on modifying a port number from binary](<https://blog.cjearls.io/2019/04/editing-executable-binary-file-with.html>)
- [Assembly instructions](<https://www.tutorialspoint.com/assembly_programming/assembly_conditions.htm>)

-----


# D) Nora
**OBJECTIVE:**
- Compile the [NoraCodes/crackmes](<https://github.com/NoraCodes/crackmes>) to binary


Download the repository containing the targets
```
$ git clone https://github.com/NoraCodes/crackmes.git
```

There's about 20 `C`-files in the repo and we have to compile them all to binary. 

I didn't feel like doing it manually one by one, so i created a quick `for loop`
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
First things first let's execute the binary and see what happens:

<img width="488" height="197" alt="Screenshot from 2026-02-06 09-38-39" src="https://github.com/user-attachments/assets/dc22c016-b22c-4e3c-b5a9-e9f2ce1a0472" />

Because I cannot look at the original source code and the machine code is alien language, I decided to exract some human readable content from the executable by using our old friend `strings`. 

#### And wouldn't you know, it delivered once again: 
<img width="457" height="541" alt="Screenshot from 2026-02-06 09-39-29" src="https://github.com/user-attachments/assets/9e0f6663-1e84-4f4a-a78b-83a3c5569b48" />


<img width="468" height="121" alt="Screenshot from 2026-02-06 09-46-25" src="https://github.com/user-attachments/assets/f2bdba7b-1685-411e-a05d-84bfdc23088a" />

-----


# E.2) crackme01e
**OBJECTIVE**
- Same as above

## Solving it
Same kind of workflow:

<img width="650" height="710" alt="Screenshot from 2026-02-06 09-50-31" src="https://github.com/user-attachments/assets/a37917f5-dae2-47f9-be33-608aebfb9656" />


### This time we had to escape a special character in order to complete the objective:

<img width="499" height="296" alt="Screenshot from 2026-02-06 09-50-44" src="https://github.com/user-attachments/assets/51b55c95-e12b-4629-b86b-4ab22cb0c734" />

-----


# F) crackme02
**OBJECTIVE**
- Name the main program's variables from the reverse-engineered binary and explain the program's operation
- Solve the binary

## Reverse Engineering, Explaining and Renaming
> I'm going to dissect the binary, rename some variables and explain what's going on as we go!


Naturally we start off by firing up the big gun a.k.a `ghidra`
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
  - If the password is correct the variable is set to `0` and returned, indicating that everything went well.
  - If the password is wrong the variable is set to `0xffffffff`, which will result in returning the highest possible exit code status.


I also found a few other variables to name. But before that, let's go through how the program works a little bit:

First of all, **the main function takes 2 parameters**
1. _Amount of arguments in the CLI command used_
  - For clarity: calling the program itself (`$ ./<program>`) counts as an argument on the CLI!
2. _Param/Input provided by the user_

If the _number of command line arguments_ are **NOT** == 2 (`program invocation` + `user input`) the program will exit.

If the arguments are exactly 2 in number, the function will then do some operation on the password and user input.


This is achieved in the following manner 

> [!NOTE]
> Code modified to drive the point across

```C
if (cliArgs == 2){
  -- doing some magic on strings --
}
else {
  puts("Exactly one argument needed");
  exitCode = FAIL;
}
```

Now let's reveal the variables before diving into the solution:

`cliArgs`, `userInput`, `exitCode` and `i` (the index used in the for loop). This makes it easier to understand the program:

<img width="969" height="540" alt="Screenshot from 2026-02-06 16-25-22" src="https://github.com/user-attachments/assets/f3370ee2-f020-4627-9e25-a48913cb11fa" />


### Now to the interesting part

The following section in the code really caught my interest:
```C
if ("password1"[i] + -1
```
It seems like the program is taking each character of the password and shifting it back by one (presumably ascii).

It also seemed like the loop is not accounting for the length of the input. Which takes us to the next step:

The letter before `p` is `o` so let's give it a go 
> It can't be wrong if it rhymes...

<img width="524" height="107" alt="Screenshot from 2026-02-06 16-45-27" src="https://github.com/user-attachments/assets/3e293b28-560f-4ceb-b655-ff1fe850d619" />


And wouldn't you know, didn't even have to type in the whole password == ``o`rrvnqc0``

Here's an [ascii table](<https://www.ascii-code.com/>) for reference. For the most part just knowing the alphabet was enough here, I just needed to know what comes before `a`, which was the backtick!
