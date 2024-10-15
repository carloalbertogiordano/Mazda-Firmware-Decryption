# Decrypting Mazda Firmware

This project documents the process of decrypting the firmware of a Mazda vehicle using techniques for decrypting encrypted archives, specifically ZIP files (or .up as Mazda calls them). The decryption was possible because the Version 74 uses **ZipCrypto** for file encryption; this algorithm is insecure.

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Preparing the firmware](#preparing-the-firmware)
- [Decryption Procedure](#decryption-procedure)
- [Final Considerations](#final-considerations)
- [Acknowledgments](#acknowledgments)

## Introduction

As a passionate hacker, my curiosity often drives me to explore the boundaries of technology. When I stumbled upon the MZD-AIO Tweaks project, I couldn't help but wonder just how far we could push the limits of what’s possible. The thrill of discovery sparked a quest within me to uncover potential vulnerabilities in the firmware of Mazda vehicles. After some diligent searching, I found the firmware available online—only to be met with a disheartening obstacle: it was password-protected.

Undeterred, I continued my exploration and came across a file extracted from the filesystem of someone’s Mazda. To my astonishment, this file was also contained within the archive I had found. What made this revelation even more exciting was the realization that the plaintext file I possessed matched the content of the archive perfectly, as evidenced by their identical CRC values. The question loomed in my mind: what encryption algorithm had they used to secure it?

After some investigation, I discovered that they had implemented **ZipCrypto** for encryption. This revelation elicited both surprise and pity; while ZipCrypto serves its purpose, it is notoriously vulnerable to attacks.

### Vulnerabilities of ZipCrypto
- **Chosen Plaintext Attack**: In a chosen plaintext attack, an adversary can encrypt plaintexts of their choosing and analyze the corresponding ciphertexts. Because ZipCrypto uses a simple stream cipher, it is susceptible to this type of attack. An attacker can craft specific inputs to deduce parts of the encryption key, eventually gaining access to the original data.
- **Known Plaintext Attack**: This vulnerability arises when an attacker has access to both the plaintext and its corresponding ciphertext. ZipCrypto’s weak encryption allows attackers to identify patterns and correlations, enabling them to recover the encryption key. Given enough samples of known plaintext and ciphertext pairs, an attacker can decrypt other files encrypted with the same key.

The use of ZipCrypto in securing critical firmware is both astonishing and concerning. It opens the door to further exploration and investigation, igniting a sense of excitement for what lies ahead.

## Prerequisites

Fortunately we don't need many programs:

- **[bkcrack](https://github.com/kimci86/bkcrack)**: This tool is essential for cracking the encryption.
- **crc32 (optional)**: This tool is useful for checking if files match. You should know how to install a package manager for your system.
- **zip**: A utility to extract the archive afterward.


## Preparing the firmwareDecrypted
Some errors arise when you try to use bkcrack with this Firmware ZIP. So let's simply change the file extension of the Firmware archive to .zip and fix a missing EOCD record error.
```
mv DownloadFirmware/cmu150_NA_74.00.230A_update.up RepairedFirmware/V74.zip
zip -FF RepairedFirmware/V74.zip --out RepairedFirmware/V74_repaired.zip
rm RepairedFirmware/V74.zip #Delete old broken file
```
## Decryption Procedure
This should be simple. We use the plain-text file we have found on-line to decipher the archive.
```
bkcrack -C RepairedFirmware/V74_repaired.zip -c jci_subord_cert.pem -p KnownFile/jci_subord_cert.pem
```
The output should contain the keys: ``a86725b0 46a06855 20ac3277``. Now let's re-zip the archive with a password we chose.
```
bkcrack -C RepairedFirmware/V74_repaired.zip -k a86725b0 46a06855 20ac3277 -U Decrypted/v74_dec.zip NEWPASSWORDHERE
```

Voilà, you can now get the content of the archive in this way:
```
unzip Decrypted/v74_dec.zip -d Decrypted #Save firmware content in Decrypt folder
```

## Final Considerations
So, what have we learned today? ZipCrypto is not a secure algorythm and should not be used.
I hope this achivement will bring forward the homebrewing comunity and support the development of new third party Mazda apps.

# Acknowledgments
I would like to express my heartfelt gratitude to everyone who contributed to this project (basically me and my multiple personalities).

- **MZD-AIO Community**: A special thank you to the MZD-AIO community for their innovative work and shared knowledge, which inspired me to delve deeper into the Mazda firmware and explore its potential vulnerabilities.
- **Open Source Contributors**: I also want to acknowledge the numerous open-source contributors whose libraries and tools made this journey possible, especially those who developed **bkcrack** 
-  **Acceis.fr**: For a simple and pragmatic [tutorial](https://www.acceis.fr/cracking-encrypted-archives-pkzip-zip-zipcrypto-winzip-zip-aes-7-zip-rar/) on the subject.
- **ChatGPT**: You can f\*\*\*off you useless piece of s\*\*t.
