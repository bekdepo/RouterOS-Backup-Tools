# RouterOS-Backup-Tools
Tools to decrypt/encrypt RouterOS v6.13+ backup files

# Usage examples  

### Info
`./ROSbackup.py info -i MikroTik.backup`  

### Decrypt  
Convert an encrypted backup to a plaintext backup  
`./ROSbackup.py decrypt -i MikroTik-encrypted.backup -o MikroTik-plaintext.backup -p password`  


### Encrypt  
Convert a plaintext backup to an encrypted backup  
`./ROSbackup.py encrypt -i MikroTik-plaintext.backup -o MikroTik-encrypted.backup -p password`  


# Header structure
## Plaintext version
| Size (byte)  | Type | Name | Description |
| :----------: | ---- | ---- | ------- |
| 4 | Unsigned LE Int | Magic | 0xB1A1AC88 |
| 4 | Unsigned LE Int | File size | length in bytes |

## Encrypted version
| Size (byte)  | Type | Name | Description |
| :----------: | ---- | ---- | ------- |
| 4 | Unsigned LE Int | Magic | 0x7291A8EF |
| 4 | Unsigned LE Int | File size | length in bytes |
| 32 | Byte array | Salt | random salt added to password |
| 4 | Unsigned LE Int | Magic check | Encrypted Magic 0xB1A1AC88 to verify if password is correct |

# Body structure
In the body are saved all file pair with extension .idx and .dat inside /flash/rw/store/  
For each file:  

| Size (byte)  | Type | Name | Description |
| :----------: | ---- | ---- | ------- |
| 4 | Unsigned LE Int | Filename length | Filename length without extension (.idx .dat) |
| Filename length | String | Filename | String without null byte terminator (and without extension .idx .dat)|
| 4 | Unsigned LE Int | IDX File size | length in bytes |
| IDX File size | Byte array | IDX File | content of IDX file |
| 4 | Unsigned LE Int | DAT File size | length in bytes |
| DAT File size | Byte array | DAT File | content of DAT file |

# Encryption setup
1) A random salt of 32 byte is generated
2) The password is appended to the salt
3) salt+password are hashed with SHA1 algorithm
4) RC4 cipher is initialized with SHA1 digest
5) RC4 cipher skip first 0x300 (256 * 3 = 768) iteration
6) 0xB1A1AC88 is encrypted to check if password is correct before performing a decryption

# Dependences
- argparse
- pycrypto
