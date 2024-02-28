# AES-128 Implementation in C++

This project is an implementation of the Advanced Encryption Standard (AES) with a block size of 128 bits (AES-128).
The implementation follows the standard as defined in the FIPS-197 publication. This encryption algorithm is widely used to protect sensitive data in various applications, including secure communication, file encryption, and password protection.
## Features

- Full implementation of AES-128 encryption and decryption in C++.
- Compliance with the FIPS-197 standard.
- Support for key expansion and round functions.
- Example of encryption and decryption in the `main()` function.

## Getting Started

To get started with this project, follow these steps:

1. Clone the repository or download the source code.
2. Ensure you have a C++ compiler installed, such as GCC or Clang.
3. Compile the source code to generate an executable file.
4. Run the executable to see the encryption and decryption examples.

## Description of the Advanced Encryption Standard algorithm

AES is an iterated block cipher with a fixed block size of 128 and a
variable key length. The different transformations operate on the
intermediate results, called *state*. The state is a rectangular array
of bytes and since the block size is 128 bits, which is 16 bytes, the
rectangular array is of dimensions 4x4. (In the Rijndael version with
variable block size, the row size is fixed to four and the number of
columns vary. The number of columns is the block size divided by 32 and
denoted Nb). The cipher key is similarly pictured as a rectangular array
with four rows. The number of columns of the cipher key, denoted Nk, is
equal to the key length divided by 32.

![image](https://user-images.githubusercontent.com/1549028/234524844-c4aabe3b-eab8-4897-82f6-28abf2ec1b5d.png)

``` 
A state:
-----------------------------
| a0,0 | a0,1 | a0,2 | a0,3 |
| a1,0 | a1,1 | a1,2 | a1,3 |
| a2,0 | a2,1 | a2,2 | a2,3 |
| a3,0 | a3,1 | a3,2 | a3,3 |
-----------------------------

A key:
-----------------------------
| k0,0 | k0,1 | k0,2 | k0,3 |
| k1,0 | k1,1 | k1,2 | k1,3 |
| k2,0 | k2,1 | k2,2 | k2,3 |
| k3,0 | k3,1 | k3,2 | k3,3 |
-----------------------------
```

It is very *important* to know that the cipher input bytes are mapped
onto the the state bytes in the order a0,0, a1,0, a2,0, a3,0, a0,1,
a1,1, a2,1, a3,1 \... and the bytes of the cipher key are mapped onto
the array in the order k0,0, k1,0, k2,0, k3,0, k0,1, k1,1, k2,1, k3,1
\... At the end of the cipher operation, the cipher output is extracted
from the state by taking the state bytes in the same order. AES uses a
variable number of rounds, which are fixed: A key of size 128 has 10
rounds. A key of size 192 has 12 rounds. A key of size 256 has 14
rounds. During each round, the following operations are applied on the
state:

1.  SubBytes: every byte in the state is replaced by another one, using
    the Rijndael S-Box
2.  ShiftRow: every row in the 4x4 array is shifted a certain amount to
    the left
3.  MixColumn: a linear transformation on the columns of the state
4.  AddRoundKey: each byte of the state is combined with a round key,
    which is a different key for each round and derived from the
    Rijndael key schedule

In the final round, the MixColumn operation is omitted. The algorithm
looks like the following (pseudo-C):

```c 
AES(state, CipherKey)
{
    KeyExpansion(CipherKey, ExpandedKey);
    AddRoundKey(state, ExpandedKey);
    for (i = 1; i < Nr; i++)
    {
        Round(state, ExpandedKey + Nb*i);
    }
    FinalRound(state, ExpandedKey + Nb * Nr);
}
```

![image](https://user-images.githubusercontent.com/1549028/234524395-752bb3be-813c-40ff-8dd0-f0a02029d229.png)


## Key Expansion

The key expansion process in AES is a mechanism to generate all the round keys needed for encryption and decryption. The process involves the following steps:

1. The user-provided key is first copied into the first round key.
2. The key expansion process then iteratively applies a series of transformations to the previous round key to generate the next round key. These transformations include rotations, substitutions, and XOR operations.
3. The process continues until all the required round keys have been generated.

The `keyExpansion` function performs the key expansion process. It takes as input the user-provided key and an array to store the generated round keys. The function iteratively applies the transformations to the previous round key to generate the next round key.

Here is an example of how to use the `keyExpansion` function:

```cpp
// Define the key as a string of normal characters
const char *keyString = "MySecretKey12345";

// Convert the key string to a uint8_t array
uint8_t key[BLOCK_SIZE];
for (size_t i = 0; i < BLOCK_SIZE; ++i) {
    key[i] = static_cast<uint8_t>(keyString[i]);
}

// Generate round keys from the user key
uint8_t roundKeys[11][BLOCK_SIZE];
keyExpansion(key, roundKeys);
```
## AES operations: SubBytes, ShiftRow, MixColumn and AddRoundKey

### The AddRoundKey operation:

In this operation, a Round Key is applied to the state by a simple
bitwise XOR. The Round Key is derived from the Cipher Key by the means
of the key schedule. The Round Key length is equal to the block key
length (=16 bytes).

``` 
-----------------------------       -----------------------------   -----------------------------
| a0,0 | a0,1 | a0,2 | a0,3 |       | k0,0 | k0,1 | k0,2 | k0,3 |   | b0,0 | b0,1 | b0,2 | b0,3 |
| a1,0 | a1,1 | a1,2 | a1,3 |  XOR  | k2,0 | k2,1 | k2,2 | k2,3 | = | b2,0 | b2,1 | b2,2 | b2,3 |
| a2,0 | a2,1 | a2,2 | a2,3 |       | k1,0 | k1,1 | k1,2 | k1,3 |   | b1,0 | b1,1 | b1,2 | b1,3 |
| a3,0 | a3,1 | a3,2 | a3,3 |       | k3,0 | k3,1 | k3,2 | k3,3 |   | b3,0 | b3,1 | b3,2 | b3,3 |
-----------------------------       -----------------------------   -----------------------------

where: b(i,j) = a(i,j) XOR k(i,j)
```

A graphical representation of this operation can be seen below:

![image](https://user-images.githubusercontent.com/1549028/234528702-5610357c-50d2-4090-b8e8-0dab12024470.png)

### The ShiftRow operation:

In this operation, each row of the state is cyclically shifted to the
left, depending on the row index.

-   The 1st row is shifted 0 positions to the left.
-   The 2nd row is shifted 1 positions to the left.
-   The 3rd row is shifted 2 positions to the left.
-   The 4th row is shifted 3 positions to the left.

``` 
-----------------------------    -----------------------------
| a0,0 | a0,1 | a0,2 | a0,3 |    | a0,0 | a0,1 | a0,2 | a0,3 |
| a1,0 | a1,1 | a1,2 | a1,3 | -> | a1,1 | a1,2 | a1,3 | a1,0 |
| a2,0 | a2,1 | a2,2 | a2,3 |    | a2,2 | a2,3 | a2,0 | a2,1 |
| a3,0 | a3,1 | a3,2 | a3,3 |    | a3,3 | a3,0 | a3,1 | a3,2 |
-----------------------------    -----------------------------
```
A graphical representation of this operation can be found below:

![image](https://user-images.githubusercontent.com/1549028/234527634-c85f1b7b-6978-4f2e-9006-b7dec501b0e6.png)

Please note that the inverse of ShiftRow is the same cyclically shift
but this time to the right. It will be needed later for decoding.

### The SubBytes operation:

The SubBytes operation is a non-linear byte substitution, operating on
each byte of the state independently. The [substitution table
(S-Box)](http://en.wikipedia.org/wiki/Rijndael_S-box) is invertible and
is constructed by the composition of two transformations:

1.  Take the multiplicative inverse in [Rijndael's finite
    field](http://en.wikipedia.org/wiki/Finite_field_arithmetic)
2.  Apply an affine transformation which is documented in the Rijndael
    documentation.

Since the S-Box is independent of any input, pre-calculated forms are
used, if enough memory (256 bytes for one S-Box) is available. Each byte
of the state is then substituted by the value in the S-Box whose index
corresponds to the value in the state:

```c 
a(i,j) = SBox[a(i,j)]
```

![image](https://user-images.githubusercontent.com/1549028/234528874-4cafa80c-01c0-4d59-a0a4-feb76e5243b2.png)

Please note that the inverse of SubBytes is the same operation, using
the inversed S-Box, which is also precalculated.

### The MixColumn operation:

I will keep this section very short since it involves a lot of very
advance mathematical calculations in the [Rijndael's finite
field](http://en.wikipedia.org/wiki/Finite_field_arithmetic). All you
have to know is that it corresponds to the matrix multiplication with:

``` 
2 3 1 1
1 2 3 1
1 1 2 3
3 1 1 2
```

and that the addition and multiplication operations are a little
different from the normal ones.

You can skip this part if you are not interested in the math involved:

> Addition and Substraction:
>
> Addition and subtraction are performed by the exclusive or operation.
> The two operations are the same; there is no difference between
> addition and subtraction.
>
> Multiplication in Rijndael's galois field is a little more
> complicated. The procedure is as follows:
>
> -   Take two eight-bit numbers, a and b, and an eight-bit product p
>
> -   Set the product to zero.
>
> -   Make a copy of a and b, which we will simply call a and b in the
>     rest of this algorithm
>
>
>     Run the following loop eight times:
>
>         1.  If the low bit of b is set, exclusive or the product p by
>             the value of a
>         2.  Keep track of whether the high (eighth from left) bit of a
>             is set to one
>         3.  Rotate a one bit to the left, discarding the high bit, and
>             making the low bit have a value of zero
>         4.  If a's hi bit had a value of one prior to this rotation,
>             exclusive or a with the hexadecimal number 0x1b
>         5.  Rotate b one bit to the right, discarding the low bit, and
>             making the high (eighth from left) bit have a value of
>             zero.
>
> -   The product p now has the product of a and b
>
![image](https://user-images.githubusercontent.com/1549028/234528985-b6a569ea-9ba0-4897-b803-16919738b811.png)

## Encryption and Decryption
After generating the round keys, the aes_encrypt and aes_decrypt functions can be used to encrypt and decrypt the plaintext message, respectively. These functions take as input the plaintext message, the user key, and the round keys generated by the keyExpansion function.

The aes_encrypt function performs the AES-128 encryption process, while the aes_decrypt function performs the AES-128 decryption process. Both functions return the ciphertext or the decrypted text, respectively.

Here is an example of how to use the aes_encrypt and aes_decrypt functions:

```cpp
void aes_encrypt(uint8_t * plainText,uint8_t * cipherText,uint8_t * userKey){
    uint8_t roundKeys[176];
    //perform KeyExpansion

    keyExpansion(userKey,roundKeys);
    
    memcpy(cipherText, plainText, BLOCK_SIZE);
    addRoundKey(cipherText,roundKeys); //Initial Round

    
    for(int i=1;i<ROUNDS;i++){// Nine Rounds
        subBytes(cipherText);
       
        shiftRows(cipherText);
        
        mixColumns(cipherText);

        addRoundKey(cipherText,roundKeys+i*BLOCK_SIZE);
        
    }

    subBytes(cipherText);
    shiftRows(cipherText);  //Final Round
    addRoundKey(cipherText,roundKeys+ROUNDS*BLOCK_SIZE);

}

void aes_decrypt(uint8_t * cipher,uint8_t * key,uint8_t * plain){
   
    
    uint8_t roundKeys[176];
    //Key Expansion
    keyExpansion(key,roundKeys);


    //initial round
    
    addRoundKey(cipher,roundKeys+BLOCK_SIZE*(ROUNDS));

    for(int i=ROUNDS-1;i>0;i--){// Nine Rounds
        invSubBytes(cipher);
        
        invShiftRows(cipher);
      
        addRoundKey(cipher,roundKeys+BLOCK_SIZE*(i));
        
        invMixColumns(cipher);
       
    }
    invShiftRows(cipher);
    invSubBytes(cipher);// Final Round
    addRoundKey(cipher,roundKeys);


     memcpy(plain,cipher,BLOCK_SIZE);

}
```
## References
https://en.wikipedia.org/wiki/Advanced_Encryption_Standard

## Conclusion

In conclusion, AES-128 encryption provides robust security features by ensuring the confidentiality and integrity of sensitive data, making it a reliable solution for day to day data transfer and other encryption services.

