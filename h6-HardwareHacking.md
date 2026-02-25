# Breaking into embedded systems

**Objectives**
1. Decrypt firmware image
2. Analyse the image file
3. Extract rootfs from the dump file
4. Extract rootfs from the image file
5. Search available applications
6. Analyse and try to open the root password


# First Things First

**The decrypter**
- https://github.com/robbins/tp-link-decrypt


**TapoV3 firmware binary**
- aws s3 cp s3://download.tplinkcloud.com/firmware/Tapo_C200v3_en_1.4.2_Build_250313_Rel.40499n_up_boot-signed_1747894968535.bin Tapo_C200v4_en_1.4.2.bin --no-sign-request


**Camera dump file**
- https://hhmoodle.haaga-helia.fi/mod/resource/view.php?id=3617382


**Useful link**
- https://quentinkaiser.be/security/2025/07/25/rooting-tapo-c200/


# Execution environment

**The host**
- <img width="907" height="511" alt="2026-02-24-23:17:05" src="https://github.com/user-attachments/assets/a63fad39-ab06-4933-a8a1-c2bc71cfe727" />

**The attacker**
- <img width="764" height="331" alt="2026-02-24-23:30:24" src="https://github.com/user-attachments/assets/fe19e135-f903-4c26-98f1-ee58b841154a" />



-----



# 1 - Firmware Image Decryption
Firs we install the decrypter
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
Running make clean first deletes old temporary files and provides a fresh build.

Now we only get one error:
```bash
src/tp-link-decrypt.c:58:10: fatal error: openssl/evp.h: No such file or directory
   58 | #include <openssl/evp.h>
      |          ^~~~~~~~~~~~~~~
compilation terminated.
```


To tackle this, we download the necessary dependency (which I thought the `preinstall.sh` script would've taken care of)
```bash
$ sudo apt install libssl-dev -y
```
Then `make clean && make` again. This time no errors and we have an executable in the `./bin` dir.
Soon we'll se if it works!


**We then download the firmware image**
```bash
$ sudo apt install awscli -y
$ aws s3 cp s3://download.tplinkcloud.com/firmware/Tapo_C200v3_en_1.4.2_Build_250313_Rel.40499n_up_boot-signed_1747894968535.bin Tapo_C200v4_en_1.4.2.bin --no-sign-request
```

And lastly the `Tapo camera C200 V3` dump file. Which is located at the **schools moodle course page**, so I downloaded it on my host machine straight to the kali share and moved it to my project folder on my Kali VM:
- <img width="681" height="205" alt="2026-02-25-20:56:35" src="https://github.com/user-attachments/assets/3ee67c16-dd11-4f64-8f8e-10e93ab628c5" />
- <img width="1093" height="280" alt="2026-02-25-20:58:16" src="https://github.com/user-attachments/assets/57f60d1e-daf1-4d7b-8ac1-462d15bfae9c" />

Now we have everything set up, let's get cracking shall we!

We decrypt the firmware image using the tool we compiled like so (the commands given are relative to my directory):
```bash
$ tp-link-decrypt/bin/tp-link-decrypt Tapo_C200v4_en_1.4.2.bin 
```
<img width="1246" height="594" alt="2026-02-25-21:07:47" src="https://github.com/user-attachments/assets/8b83be8d-2833-459d-9ef1-ad4a2d35e9a5" />


We confirm that the operation was succesfull, by printing any human readable strings from the two files. We can notice that the output from the encrypted file doesn't make sense, whereas the unencrypted one is already singing to us:
- <img width="556" height="601" alt="2026-02-25-21:14:13" src="https://github.com/user-attachments/assets/b8f75a88-3355-4972-be13-9c5a0f5526ba" />
- <img width="1269" height="693" alt="2026-02-25-21:14:43" src="https://github.com/user-attachments/assets/05be6d90-4d68-432b-ac86-fd8a47ae41b6" />


----




# 2 - Analysing the image file









