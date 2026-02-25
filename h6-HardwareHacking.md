# Breaking into embedded systems

Objectives
1. Decrypt firmware image
2. Analyse the image file
3. Extract rootfs from the dump file
4. Extract rootfs from the image file
5. Search available applications
6. Analyse and try to open the root password


# First Things First

Download and install
- https://github.com/robbins/tp-link-decrypt


Download TapoV3 firmware binary
- aws s3 cp s3://download.tplinkcloud.com/firmware/Tapo_C200v3_en_1.4.2_Build_250313_Rel.40499n_up_boot-signed_1747894968535.bin Tapo_C200v4_en_1.4.2.bin --no-sign-request


Download camera dump file
- https://hhmoodle.haaga-helia.fi/mod/resource/view.php?id=3617382


Useful link
- https://quentinkaiser.be/security/2025/07/25/rooting-tapo-c200/


# Execution environment

The host
- <img width="907" height="511" alt="2026-02-24-23:17:05" src="https://github.com/user-attachments/assets/a63fad39-ab06-4933-a8a1-c2bc71cfe727" />

The operator
- <img width="764" height="331" alt="2026-02-24-23:30:24" src="https://github.com/user-attachments/assets/fe19e135-f903-4c26-98f1-ee58b841154a" />



# 1 - Decrypt Firmware Image
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

I don't want to focus on debugging this for hours on end, so I take the help of a little friend:
- <img width="978" height="671" alt="2026-02-25-16:17:28" src="https://github.com/user-attachments/assets/6b17a712-a90d-4c54-a506-5055df1b7a7b" />


We fix it and run make again, only to get the following in return
```bash
gcc -Wno-implicit-function-declaration -Isrc -c src/md5/md5.c -o obj/md5/md5.o
src/md5/md5.c: In function ‘MD5_Init’:
src/md5/md5.c:105:6: warning: old-style function definition [-Wold-style-definition]
  105 | void MD5_Init (mdContext)
      |      ^~~~~~~~
src/md5/md5.c: In function ‘MD5_Update’:
src/md5/md5.c:122:6: warning: old-style function definition [-Wold-style-definition]
  122 | void MD5_Update (mdContext, inBuf, inLen)
      |      ^~~~~~~~~~
src/md5/md5.c:124:16: error: argument ‘inBuf’ doesn’t match prototype
  124 | unsigned char *inBuf;
      |                ^~~~~
In file included from src/md5/md5.c:37:
src/md5/md5.h:65:6: error: prototype declaration
   65 | void MD5_Update(MD5_CTX *ctx, const unsigned char *data, unsigned int len);
      |      ^~~~~~~~~~
src/md5/md5.c:151:7: error: too many arguments to function ‘Transform’; expected 0, have 2
  151 |       Transform (mdContext->buf, in);
      |       ^~~~~~~~~  ~~~~~~~~~~~~~~
src/md5/md5.c:51:13: note: declared here
   51 | static void Transform ();
      |             ^~~~~~~~~
src/md5/md5.c: In function ‘MD5_Final’:
src/md5/md5.c:160:6: warning: old-style function definition [-Wold-style-definition]
  160 | void MD5_Final (hash, mdContext)
      |      ^~~~~~~~~
src/md5/md5.c:186:3: error: too many arguments to function ‘Transform’; expected 0, have 2
  186 |   Transform (mdContext->buf, in);
      |   ^~~~~~~~~  ~~~~~~~~~~~~~~
src/md5/md5.c:51:13: note: declared here
   51 | static void Transform ();
      |             ^~~~~~~~~
src/md5/md5.c: In function ‘Transform’:
src/md5/md5.c:203:13: warning: old-style function definition [-Wold-style-definition]
  203 | static void Transform (buf, in)
      |             ^~~~~~~~~
make: *** [Makefile:26: obj/md5/md5.o] Error 1
```






