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






