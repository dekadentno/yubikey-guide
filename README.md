# yubikey-guide
Simple steps on how to configure Yubico YubiKey 4 on Linux for signed git commits

## Install gpg2 
__Important:__ Do not use gpg because YubiKey 4 works only with gpg2 !

## Generate and export new gpg2 key
__Important:__ use RSA2048, not RSA4086!

Generate new key and answer all questions:
```bash
# generate new key
gpg2 --gen-key
```

```bash
# list generated key
gpg2 --list-secret-keys --keyid-format LONG
```
The command above will return something like this:
```bash
/home/username/.gnupg/secring.gpg
-------------------------------
sec   4096R/<COPY_LONG_KEY> 2016-08-11 [expires: 2018-08-11]
uid                          User Name <user.name@email.com>
ssb   4096R/62E5B29EEA7145E 2016-08-11

```

Export public key: 
```bash
gpg2 --armor --export PASTE_LONG_KEY_HERE > public-key.txt
```

Export private key:
```bash
gpg2 --armor --export-secret-keys PASTE_LONG_KEY_HERE > private-key.txt
```

## Import public key to Github/Gitlab
Go to your profile nad settings and find GPG keys. Import or paste the secret key there and save it.

## Tell git to use gpg2 and autosign the commits

```bash
git config --global user.signingkey YOUR_SHORT_KEY
git config --global commit.gpgsign true 
git config --global gpg.program gpg2 
```

## Transfer key to YubiKey
Every "row" is a separate command, be careful:

```bash
gpg2 --edit-key YOUR_SHORT_KEY

toggle

keytocard

choose 1 for signature key and replace the existing key

key 1

keytocard
```

After last command, select (1) Signature key and add passphrase.

```bash
key 1

```


If you mess something up on your YubiKey 4, you can do a factory reset with [this](https://gist.github.com/pkirkovsky/c3d703633effbdfcb48c) script.

## Avoid entering password on every pull and push:
https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
