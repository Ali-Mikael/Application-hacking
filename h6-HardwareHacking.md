# Breaking Into Embedded Systems

**Objectives**
1. Decrypt firmware image
2. Analyse the image file
3. Extract rootfs from the dump file
4. Extract rootfs from the image file
5. Search available applications
6. Analyse and try to open the root password

# Execution Environment

**The Provider**
- <img width="907" height="511" alt="2026-02-24-23:17:05" src="https://github.com/user-attachments/assets/a63fad39-ab06-4933-a8a1-c2bc71cfe727" />

**The Attacker**
- <img width="764" height="331" alt="2026-02-24-23:30:24" src="https://github.com/user-attachments/assets/fe19e135-f903-4c26-98f1-ee58b841154a" />


# First Things First

**The decrypter**
- https://github.com/robbins/tp-link-decrypt


**TapoV3 firmware binary**
- aws s3 cp s3://download.tplinkcloud.com/firmware/Tapo_C200v3_en_1.4.2_Build_250313_Rel.40499n_up_boot-signed_1747894968535.bin Tapo_C200v4_en_1.4.2.bin --no-sign-request


**Camera dump file**
- https://hhmoodle.haaga-helia.fi/mod/resource/view.php?id=3617382


**Useful link**
- https://quentinkaiser.be/security/2025/07/25/rooting-tapo-c200/



-----



# Preparation
Installing the decrypter 

(read the [repo](<https://github.com/robbins/tp-link-decrypt>) README for more!)
```bash
$ git clone https://github.com/robbins/tp-link-decrypt.git
$ cd tp-link-decrypt
$ ./preinstall.sh (downloading dependencies)
$ ./extract_keys.sh (extract RSA/DES keys from TP-Link Firmware)
$ rm -rf tmp.fwextract (remove temporary directory)
$ make
```

The last command resulted in a bunch of errors:
- <img width="1357" height="707" alt="2026-02-24-23:44:55" src="https://github.com/user-attachments/assets/09a4fd4c-b233-42f3-b624-32f0f4c12cad" />

I don't want to focus on debugging this for hours on end, so I messaged a [friend](<https://github.com/MatPohj>), who also had the same issue as me. He came in clutch. Shoutout to Matti! 🙏


The problem is that the code syntax is outdated, so we have to add the `-std=gnu89` flag to the `Makefile` like this:
```bash
$ vim Makefile
>>>
CFLAGS = -std=gnu89 -Wno-implicit-function-declaration -I$(SRCDIR)
<<<
```

Then
```bash
$ make clean
$ make
```
Running make clean first, deletes old temporary files and provides a fresh build

Now we only get one error:
```bash
src/tp-link-decrypt.c:58:10: fatal error: openssl/evp.h: No such file or directory
   58 | #include <openssl/evp.h>
      |          ^~~~~~~~~~~~~~~
compilation terminated.
```


To tackle this, we download the necessary dependency
```bash
$ sudo apt install libssl-dev -y
$ make clean
$ make
```
This time no errors and we have the executable in the `./bin` dir!


We then install `awscli` in order to download the firmware image from an `S3 bucket`:
```bash
$ sudo apt install awscli -y
$ aws s3 cp s3://download.tplinkcloud.com/firmware/Tapo_C200v3_en_1.4.2_Build_250313_Rel.40499n_up_boot-signed_1747894968535.bin Tapo_C200v4_en_1.4.2.bin --no-sign-request
```


And lastly the `Tapo camera C200 V3` dump file. Which is located at the schools **moodle course page**, so I downloaded it from my host machine straight to the `kali share` and moved it to my project folder at the destination:
- <img width="681" height="205" alt="2026-02-25-20:56:35" src="https://github.com/user-attachments/assets/3ee67c16-dd11-4f64-8f8e-10e93ab628c5" />
- <img width="1093" height="280" alt="2026-02-25-20:58:16" src="https://github.com/user-attachments/assets/57f60d1e-daf1-4d7b-8ac1-462d15bfae9c" />

Now that everything is set up nicely, let's get cracking shall we!



-----



# Decrypting the Firmware Image

Now we get to use the tool we compiled earlier: 
```bash
$ tp-link-decrypt/bin/tp-link-decrypt Tapo_C200v4_en_1.4.2.bin 
```
Note that the command given is relative to my working directory!

<img width="1246" height="594" alt="2026-02-25-21:07:47" src="https://github.com/user-attachments/assets/8b83be8d-2833-459d-9ef1-ad4a2d35e9a5" />


We confirm that the operation was successful by printing any human readable strings from the two files. We can notice that the output from the encrypted file doesn't make sense (1st picture), whereas the **unencrypted** one (2nd picture) is already revealing secrets to us:
- <img width="556" height="601" alt="2026-02-25-21:14:13" src="https://github.com/user-attachments/assets/b8f75a88-3355-4972-be13-9c5a0f5526ba" />
- <img width="1269" height="693" alt="2026-02-25-21:14:43" src="https://github.com/user-attachments/assets/05be6d90-4d68-432b-ac86-fd8a47ae41b6" />


----




# Analysing The Image & Extracting The Rootfs
Now to the meat & bones.

We analyse the file using the `binwalk` utility.


**Q:** What is binwalk?

**A:** Binwalk is a tool for searching binary images for embedded files and executable code. It's designed for identifying files and code embedded inside of firmware images. >[source](<https://www.kali.org/tools/binwalk/>)<



```bash
$ binwalk Tapo_C200v4_en_1.4.2.bin.dec
```
<img width="1371" height="652" alt="2026-02-25-22:25:37" src="https://github.com/user-attachments/assets/0f901ea9-95f5-41e2-9358-13cf22b3e6cb" />


I noted the following from the output produced by binwalk. -->

The image contains atleast:
- **Android bootimage**
- **uImage header** which describes the following:
  - Size: 1288473 bytes
  - OS: Linux
  - CPU type: MIPS
  - Compressions type: lzma
  - Image Name: mips Ingenic Linux-3.10.14"
- **Squashfs** filesystem
  - xz compressed
  - 3032084 bytes in size
  - little endian

The Squashfs filesystem is the main target at the moment. That's where we'll head next.

**Q:** What is squashfs?

**A:** _A compressed read-only filesystem for Linux_. Intended (among other things) for embedded systems where low overhead is needed! >[Source](<https://docs.kernel.org/filesystems/squashfs.html>)<


Now that we have some sort of grasp what the image file contains, let's take `binwalk` for a spin again and **extract the contents** using the `--extract` flag 
```bash
$ binwalk -e Tapo_C200v4_en_1.4.2.bin.dec
```
<img width="1365" height="315" alt="2026-02-25-22:32:21" src="https://github.com/user-attachments/assets/7fcf1ddf-30c7-4164-ac6f-bf1e6fac4bbe" />


Time to collect:
```bash
$ cd _Tapo_C200v4_en_1.4.2.bin.dec.extracted 
$ ls
$ cd squashfs-root
$ ls
```
<img width="1318" height="246" alt="2026-02-25-22:33:27" src="https://github.com/user-attachments/assets/612caa0f-2689-495e-9491-6e349ffac1ad" />

This was a nice find, as we aquired the root filesystem including `main program` which we can now dissect with tools such as `strings`, `objdump`, `ghidra`, `gdb` and the likes.
- <img width="1360" height="222" alt="2026-02-25-22:44:27" src="https://github.com/user-attachments/assets/faf6a541-3502-4a59-8555-7381a42d4d7d" />


I also roamed around the files system opening up scripts and looking for interesting executables. Trying to grasp how the IP-camera operates.

**A list of interesting entries:**
- `/bin/gdbserver`
  - Remote debugging server
  - This could be a big one if gained access to
- `/bin/main`
  - The main program obviously
- `/bin/impdbg`
  - Couldn't find much information on this online, except the AI summary on the top of the page saying this could be a debug interface / internal diagnostics. Go and figure
- `/usr/sbin/hostapd`
  - User space daemon for APs and authN servers
  - Found info on it [here](<https://community.infineon.com/t5/Knowledge-Base-Articles/HostApd-setup-in-Linux/ta-p/246026#.>)


### We dissect the main program
```bash
$ strings main | egrep -i 'admin|pass|root|pwd|user|login|auth|username'
```

This give us a whole bunch of hits, `381` to be exact. So it looks like we're digging in the right spot. 


My next step was to gain some context around the hits. This is achieved for example by printing 5 lines before & after the target, like so:
```
$ strings main | grep -i -C 5 passw
```


I did the same for admin, pwd and a few others. A few interesting entries:
- `http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordDigest`
- `digest_password`
- `gen_root_passwd`
- `/www/cert/onvif/private_key.pem`
- `wep_key0`

<img width="1429" height="516" alt="2026-02-26-00:24:30" src="https://github.com/user-attachments/assets/c2725166-cf5f-4d3c-a440-c61456c2782e" />

<img width="828" height="330" alt="2026-02-26-17:07:24" src="https://github.com/user-attachments/assets/a203fdde-1ff1-4b1b-a740-ded876b2b7a7" />

<img width="832" height="532" alt="2026-02-26-17:10:10" src="https://github.com/user-attachments/assets/70bc7ead-e406-4f4a-add7-59f8eb4d10b7" />


We can also note from the output that the `ONVIF` service is in play

**Q:** Yes but what is is?

**A:** ONVIF "provides standardized interfaces for effective interoparability of IP-based physical security products and services". >[source](<https://www.onvif.org/>)<


**Q:** Why are we noting this down?

**A:** Because it will better help us understand the system we're working with. It also gives us clues as where to look when hunting for the password!


I then tried to disassemble main using `objdump`, but got the following error:
```bash
$ objdump -d main

main:     file format elf32-little

objdump: can't disassemble for architecture UNKNOWN!
```

So we open up Ghidra for further examination. I'm already set on finding the `gen_root_passwd` function for starters.



------


# Hunting The Root Password
We create a new project in `Ghidra`, import the main program, and let Ghidra analyse it for us.


I started going through the `symbol tree` looking for functiongs that might reveal more secrets to us. 

My first find was: `strcmp`. Why? Because it could lead us to parts in the code where authentication logic is handled. I opened it up and found all the references to it:

<img width="1096" height="509" alt="2026-02-26-23:28:26" src="https://github.com/user-attachments/assets/2356671e-f97c-4b13-986e-e490f9605362" />


I went through all of them and renamed them to the best of my ability:

<img width="433" height="480" alt="2026-02-27-01:44:08" src="https://github.com/user-attachments/assets/a21bc51b-8da7-4259-bfd0-500ea97cfcd8" />

I found a lot of stuff in the process, here's one block for example:

```C
    uVar1 = FUN_0064d1b0(param_1);
    FUN_0052e07c(iVar7,
                 "HTTP/1.0 401 Unauthorized\r\nServer: Streamd\r\nDate: %s\r\nPragma: no-cache\r\nCa che-Control: no-cache\r\nContent-Length: 0\r\nWWW-Authenticate: Digest realm=\"%s\" ,algorithm=\"MD5\",encrypt_type=\"3\",qop=\"auth\",nonce=\"%s\",opaque=\"%s\"\r\nCo nnection: close\r\n\r\n"
                 ,uVar1,&DAT_007e8770,param_1 + 0x4318,"64943214654649846565646421");
    FUN_006499c4(param_1,0);
    *(undefined4 *)(param_1 + 0x4c) = 0xffffffff;
```
The **number string** was intriguing, but I didn't get any further with it for the time being.

We could also note the use of the `MD5 algorithm` in conjuction with a `nonce` ( a unique value used once when encrypting ).


I then opened the `Defined Strings` section side by side in Ghidra and started typing in stuff I found from the strings output earlier. 

When we locate a string:
- Double click it
- Ghidra takes us to the section in the instructions where we see all the references to it
- We then go through the reference(s)
- By double clicking the reference Ghidra will display the C-pseudocode

Here's a list I went through:
- gen_root_passwd
- digest
- loginPwd
- admin
- root
- password
- md5

### Example workflow:
<img width="1018" height="870" alt="2026-02-27-00:39:31" src="https://github.com/user-attachments/assets/87556c8c-893b-422c-8229-8f1b55848187" />

<img width="946" height="196" alt="2026-02-27-02:12:31" src="https://github.com/user-attachments/assets/99910e95-25ca-4013-b415-969760438c3d" />

<img width="967" height="654" alt="2026-02-27-02:13:03" src="https://github.com/user-attachments/assets/e31b4abb-af36-4bbd-9c36-ecdc33fb8f04" />



Found some pretty interesting functions. Here's one as an example:

I already renamed the function and a few variables that made sense to me:
```C

int get_hub_password_list(undefined4 json_hub_obj,int *hub_password_table)

{
  int hub_password_array;
  int encrypt_type_result;
  char *password_str;
  int iVar1;
  int result;
  int i;
  int iVar2;
  int *password_buffer;
  int encrypt_type;
  
  encrypt_type = 0;
  hub_password_array = jso_obj_get(json_hub_obj,"hub_password_list");
  if (hub_password_array == 0) {
    result = -0x9d0d;
  }
  else {
    i = jso_array_length(hub_password_array);
    iVar2 = 4;
    if (i < 5) {
      iVar2 = i;
    }
    for (i = 0; i < iVar2; i = i + 1) {
      encrypt_type_result = jso_array_get_idx(hub_password_array,i);
      password_str = (char *)jso_obj_get_string_origin(encrypt_type_result,"password");
      password_buffer = hub_password_table + 1;
      if (password_str == (char *)0x0) {
        return -0x9d11;
      }
      strncpy((char *)password_buffer,password_str,0x40);
      iVar1 = jso_obj_get_int(encrypt_type_result,"encrypt_type",&encrypt_type);
      if (iVar1 == -1) {
        return -0x9d11;
      }
      *hub_password_table = encrypt_type;
      hub_password_table = hub_password_table + 0x12;
      msg_debug(0,0x13,2,"get_passwd_list",0x217,"[HUB_MANAGE]get passwd:%s, encrypt_type: %d",
                password_buffer,encrypt_type);
    }
    result = 0;
  }
  return result;
}
```
It would seem as the program is storing the hub_password_list as a json object, and accessing it that way.


At this point I had spent 3 hours in Ghidra going from one place to another trying to make sense of it all, and it was already 2am...

We'll continue fresh in the morning!


### 1 day later (not in fact, the next morning.. 😂)
During my time of absence, I had an idea.

I'd been so focused on main, I didn't give the other files from the _extracted image_ enough attention. 

<img width="1465" height="460" alt="2026-02-28-17:07:07" src="https://github.com/user-attachments/assets/772bcc4b-3cb5-462f-b09f-39140843139f" />


There were so many of them tho, and as you already know, if we can avoid manual labour we will, so I created a quick shell loop
```bash
for f in ??????; do
  echo "------ $f ------" >> results.txt
  strings -n 10 $f | egrep -i 'passw|pwd|admin|root' >> results.txt
done
```
Explanation:
- `??????` will match all file names with exactly 6 characters
- The `echo` is there to pinpoint which file we got the information from
- We then extract all human readable strings (minimum 10 chars in length)
- Pipe it to `egrep` (same as `grep -E`) and do a case insensitive search for the words specified
- And lastly append the results to a text file we can refer to later


The whole operation took about 1 second. 

We then start combing through the file:
```bash
$ less results.txt
```
<img width="1484" height="308" alt="2026-02-28-17:26:03" src="https://github.com/user-attachments/assets/d1e8433e-532e-47dc-96a0-2f84ee46e73c" />


I was quite baffled to be honest. The admin password was just chilling there as I opened up my `results.txt` file. (the ones who were paying attention realised this is the same string we already caught earlier)

Last time I spent 4 hours trying to find the password and now when I came back I found it in 10 minutes, which of the majority of time was spent on opening up my machine and starting the VM.


Time and time again i'm reminded that breaks are actually good for you! 😂 💯



> [!NOTE]
> 
> It does seem like the `passwd` we extracted is likely a hash.
>
> Even so, this is huge progress. And if I have enough time, I will try to crack the hash. 
>
> And if nothing else, we could just reset the camera and input the default factory password. Maybe?
>
> Or just try to circumvent the whole hashing function and try to compare the password directly?




------



# Extracting the rootfs from the dump file

```bash
$ binwalk -e dump-tapo-c200v3-1.4.2.bin
$ cd _dump-tapo-c200v3-1.4.2.bin.extracted
$ ls
```
<img width="1751" height="644" alt="2026-02-26-22:29:20" src="https://github.com/user-attachments/assets/7afdb7c3-2641-47f9-9824-ca33780e722d" />


