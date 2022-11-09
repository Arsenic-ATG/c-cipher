# C-CIPHER

A program to encrypt/decrypt a file using the **AES-CTR-256** cipher.

It can be used to store passwords (or any other kind of sensible information) 
locally, in a private way.  
The user only needs one password to both encrypt and decrypt the information.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

## Details on Security

The algorithm is implemented in C, using the [OpenSSL]'s Encryption [API].

The **Encryption Key** has to be 256-bits long; it is generated by computing
`SHA256(p)`, with `p` being the user's password.<sup>[1](#f1)</sup>  
The same key is used for both encryption and decryption.

The _Initialization Vector IV_ is a string of `16` bytes.  
The last (**LSB**)<sup>[2](#f2)</sup> `8` bytes contain a counter which will be
incremented at every new encrypted block.  
The first (**MSB**) `8` bytes contain the "_nonce_": a random sequence of bits.  
When encrypting, the _nonce_ is filled with random bytes; these will be
prepended to the ciphertext in order to read them during the decryption
procedure.

The _block length n_, is an important parameter for determining the concrete
security of the cipher.  
Being the **IV** a uniform string of _n-bits_, it is expected to repeat after
having performed "only" `2`<sup>`n/2`</sup> encryptions.<sup>[3](#f3)</sup>  
In our case `n` is `128` bits; this means that only `2`<sup>`64`</sup> blocks -
or `2*10`<sup>`5`</sup> **Petabytes** - can be safely encrypted without ever
changing the `IV`.  
This is more than safe for a regular text file.

### Tampering

It must be noted that this mode of encryption provides no security against data
tampering.  
If the ciphertext were to be modified even by just one bit, the *integrity* of
the information would be lost.

The scope of this project however is to provide a trivial example of encryption,
to be used for simpler "protection" against *adversaries* who know little to
nothing about cryptography.

<br>

<sup id="f1">**1**</sup> The usage of a **PBKDF** (Password-Based Key Derivation Function) would further improve security.  
<sup id="f2">**2**</sup> In **big-endian** order.  
<sup id="f3">**3**</sup> Similar concept to '[The Birthday Problem]'.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

## Installation

Clone or [download] this repository; then, `cd` into the project's root folder.

Compile the program (take a look at the [dependencies] listed below):
```sh
make ccipher
```

If compilation succeeds, proceed to copy the program and the script in the bin
folder:

```sh
cp bin/ccipher ~/.local/bin
cp scripts/scipher ~/.local/bin
chmod +x ~/.local/bin/*cipher
```

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

## Usage

There are two ways of using the program.

The [first] is by executing it directly so as to have total control on what it
does;  
The [second] method provides a more user-friendly experience and offers more
functionalities with the help of a shell [script].

### Manual interaction

Execute the program directly:

```sh
ccipher arg1 arg2 arg3 arg4
```

The command line arguments needed are, in order:

+ input file (to apply the cipher on)
+ output file
+ mode (either '**encrypt**' or '**decrypt**')
+ password

_**Note:** using this approach means that anyone with access to the computer
can learn the password since it is stored in the shell's history.
A workaround might be requesting the 4 arguments **inside** the program but that
would require additional checks._

#### Example

The first time you run the program the input file should be the plain text file
containing the private information.  
The output file will store the encrypted version of it.

It is at this point that you must choose the **password** to use. It can contain
any ASCII character and can be of any (reasonable) length.

```sh
ccipher /path/to/plain.txt /path/to/private.dat encrypt myPassword123
```

The following times, the plaintext version can be recreated with the command:

```sh
ccipher /path/to/private.dat /path/to/plain2.txt decrypt myPassword123
```

Notice how the two paths have been swapped, the mode is now **de**crypt and the
password remains the same.

### Wrapper script

An easier way to interact with a private file using `ccipher` is by means of
the provided shell script: `scipher`.  
It acts as a wrapper for the main program, both to hide any sensible information
and to provide more functionalities.

For example, it automatically handles the creation and destruction of the plain
file, and its command is much simpler (no need to specify _mode_, _input file_,
_output file_, ect...).
```sh
scipher [OPTION]...
```

#### Options

The script only needs one (optional) command line argument. It specifies the
action to perform:

| Option      | Description |
| ----------- | ----------- |
| -h, --help    | Show help guide and exit.       |
| --hide        | Suppress terminal echo when typing in the password.       |
| -w, --write   | Edit the file.        |
| -r [STRING], --read [STRING] | Show a decrypted version of the entire file or of any lines containing **STRING**, if **STRING** is given. |
| -e, --export | Decrypt the file and save a copy of the plaintext. If you only need to take a quick look, consider using '`--read`' instead. |

#### Example

The first time, the script should be invoked with the `--write` option in order
to write (or copy) the information you want to hide in it.

```sh
scipher -w
```

The user will be prompted to choose a password.  
**BE CAREFUL NOT TO LOSE IT** or you won't be able to access the contents of the
file ever again.  
The script will then proceed to encrypt the data into a file which path can be
modified (as explained below).

#### Configuration

Inside the script there are a few parameters that can be modified:

+ **EDITOR**  
    Text editor of choice to use when editing the file.

+ **MAIN_FOLDER**, **ENCRYPTED_FILE**, **PLAINTEXT_FILE**  
    Path to the script's working directory and the files stored in it.

_**Note:** changing the paths is not recommended; if you do remember to move any
pre-existing files to their new location._
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

## Dependencies

+ gcc
+ make
+ libssl3  
    On *Ubuntu **22.04+*** based distributions:
    ```sh
    sudo apt install libssl-dev
    ``` 
    Otherwise, download [here] and follow [these] instructions to install.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

## Author

Marco Plaitano

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

## License

Distributed under the [MIT] license.


<!-- Links -->

[OpenSSL]:
https://www.openssl.org/
"Main Website"

[API]:
https://www.openssl.org/docs/man3.0/man3/EVP_aes_256_ctr.html
"Online Documentation"

[The Birthday Problem]:
https://en.wikipedia.org/wiki/Birthday_problem
"Wikipedia Article"

[download]:
https://github.com/marcoplaitano/credentials-storer/archive/refs/heads/main.zip
"ZIP Download"

[dependencies]:
#dependencies
"Anchor to header"

[first]:
#manual-interaction
"Anchor to header"

[second]:
#wrapper-script
"Anchor to header"

[script]:
scripts/scipher
"Repository file"

[here]:
https://www.openssl.org/source/

[these]:
https://github.com/openssl/openssl/blob/master/INSTALL.md

[MIT]:
LICENSE
"Repository file"
