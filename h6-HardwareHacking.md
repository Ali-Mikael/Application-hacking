# Breaking into embedded systems

**Objectives**
1. Decrypt firmware image
2. Analyse the image file
3. Extract rootfs from the dump file
4. Extract rootfs from the image file
5. Search available applications
6. Analyse and try to open the root password

# Execution environment

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


The problem is that the code syntax is outdated, so we have to add the `-std=gnu89` flag to the makefile like this:
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




# Analysing
Now to the juicy part.

We analyse the file using the `binwalk` utility.
```bash
$ binwalk Tapo_C200v4_en_1.4.2.bin.dec
```

**Q:** What is binwalk?

**A:** Binwalk is a tool for searching binary images for embedded files and executable code. It's designed for identifying files and code embedded inside of firmware images. [source](<https://www.kali.org/tools/binwalk/>)


I took up the following:
```
0x1683D         Android bootimg, kernel size: 0 bytes, kernel addr: 0x70657250, ramdisk size: 543519329 bytes, ramdisk addr: 0x6E72656B, product name: "mem boot start"


x20400         uImage header, header size: 64 bytes, header CRC: 0x8F2C487E, created: 2025-03-13 03:14:58, image size: 1299473 bytes, Data Address: 0x80010000, Entry Point: 0x80329EF0, data CRC: 0xEBBECE5F, OS: Linux, CPU: MIPS, image type: OS Kernel Image, compression type: lzma, image name: "mips Ingenic Linux-3.10.14"


0x3E0200        Squashfs filesystem, little endian, version 4.0, compression:xz, size: 3032084 bytes, 96 inodes, blocksize: 65536 bytes, created: 2025-03-13 03:15:05

```
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



-----
# Extracting the rootfs

There's a lot of compressed data in the file
- <img width="1371" height="652" alt="2026-02-25-22:25:37" src="https://github.com/user-attachments/assets/0f901ea9-95f5-41e2-9358-13cf22b3e6cb" />

So we take binwalk for a spin again and extract the contents using the `--extract` flag
```bash
$ binwalk -e Tapo_C200v4_en_1.4.2.bin.dec
```
<img width="1365" height="315" alt="2026-02-25-22:32:21" src="https://github.com/user-attachments/assets/7fcf1ddf-30c7-4164-ac6f-bf1e6fac4bbe" />



We then go hunting
```bash
$ cd _Tapo_C200v4_en_1.4.2.bin.dec.extracted 
$ ls
$ cd squashfs-root
$ ls
```
<img width="1318" height="246" alt="2026-02-25-22:33:27" src="https://github.com/user-attachments/assets/612caa0f-2689-495e-9491-6e349ffac1ad" />

**Q:** What is squashfs?

**A:** _A compressed read-only filesystem for Linux_. Intended (among other things) for embedded systems where low overhead is needed!
  - Read more about it [here](<https://docs.kernel.org/filesystems/squashfs.html>)

This was a nice find, as we aquired the `main program` which we can now dissect with tools such as `strings`, `objdump`, `ghidra`, `gdb` and the likes.
- <img width="1360" height="222" alt="2026-02-25-22:44:27" src="https://github.com/user-attachments/assets/faf6a541-3502-4a59-8555-7381a42d4d7d" />


I also roamed around the files system opening up scripts and looking for interesting executables.

A list of interesting entries:
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


We start dissecting the main program:
```bash
$ strings main | egrep -i 'admin|pass|root|pwd|user|login|auth|username'
```

This give us a whole bunch of hits, `381` to be exact. So it looks like we're digging at the right spot. My next step was finding individual hits, by printing 5 lines before & after the target, like so:
```
$ strings main | grep -i -C 5 passw
```
I did the same for admin, pwd and a few other. A few interesting entries:

<img width="1429" height="516" alt="2026-02-26-00:24:30" src="https://github.com/user-attachments/assets/c2725166-cf5f-4d3c-a440-c61456c2782e" />

<img width="814" height="265" alt="2026-02-26-00:26:24" src="https://github.com/user-attachments/assets/4f5793a6-d7c6-4ca7-8352-581f1e383a90" />

<img width="700" height="177" alt="2026-02-26-17:00:13" src="https://github.com/user-attachments/assets/35458e5b-8fdc-442d-8e02-bf5f2ef473aa" />

<img width="828" height="330" alt="2026-02-26-17:07:24" src="https://github.com/user-attachments/assets/a203fdde-1ff1-4b1b-a740-ded876b2b7a7" />

<img width="832" height="532" alt="2026-02-26-17:10:10" src="https://github.com/user-attachments/assets/70bc7ead-e406-4f4a-add7-59f8eb4d10b7" />






I tried to disassemble main using `objdump`, but got the following error:
```bash
$ objdump -d main

main:     file format elf32-little

objdump: can't disassemble for architecture UNKNOWN!
```

So we open up ghidra for further examination. I'm already set on finding the `gen_root_passwd` function for starters.



------

# Hunting the Root Password


