# A) Install Ghidra
For `kali linux`:
```
$ sudo apt update && sudo apt install -y ghidra
```
Run it by typing `$ ghidra`:

<img width="1202" height="738" alt="Screenshot from 2026-01-27 21-54-40" src="https://github.com/user-attachments/assets/38020368-4485-4cdc-b92a-e287703f46c4" />


# B) Rever-C
**Objective:**
- Reverse engineer the packd binary to C language using `ghidra`
- Find the main program
- Give variables descriptive names
- Explain the programs operation
- Solve the task from the binary, without the original source code.

Target can be installed from [here!](<https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip>)


## Reverse engineering
I created a new directory for my project
```
$ mkdir -p ghidraProjects/packd/
```
Then I fired up `ghidra` and opened up the binary. 

<img width="668" height="525" alt="Screenshot from 2026-02-04 23-00-23" src="https://github.com/user-attachments/assets/c6f4e5f5-21e1-4832-9267-7c35c2257808" />

<img width="688" height="373" alt="Screenshot from 2026-02-04 23-02-01" src="https://github.com/user-attachments/assets/07133052-f5b5-4122-a9f1-1bb1c9f10537" />


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
- if `true` == the flag is provided
- `else` == no bonus :/

------


# C) Backwards
**Objective**
- Modify the passtr programs's binary so that it accepts all passwords except the correct one
  - Without the original source code of course!
- Demonstrate with tests that your solution works

# Manipulation
We import the second program called `passtr` --> analyze it --> open up the main function --> give variables descriptive names:

<img width="1039" height="356" alt="Screenshot from 2026-02-04 23-34-26" src="https://github.com/user-attachments/assets/f12f92aa-f8fe-426c-9a35-91d15ae60d8f" />

<img width="833" height="447" alt="Screenshot from 2026-02-04 23-37-18" src="https://github.com/user-attachments/assets/ac8830d5-3d42-4e6f-a169-ae6f74be359f" />


