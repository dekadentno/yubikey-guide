# yubikey-guide
Simple steps on how to configure Yubico YubiKey 4 on Linux for signed git commits

## Install gpg2 
__Important:__ Do not use gpg because YubiKey 4 works only with gpg2 !

## Generate and export new gpg2 key
__Important:__ use RSA2048, not RSA4096!

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
List SHORT public keys
```bash
pub   rsa2048/66A72312 2018-09-04 [SC] [expires: 2020-09-03]
      E0B7C039870AEB65EB4CD59F5CFD524D66A72312
uid         [ultimate] Matej Lazarevic <matej.lazarevic@33barrage.com>
sub   rsa2048/<COPY_SHORT_KEY> 2018-09-04 [E] [expires: 2020-09-03]
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

## Import key to another computer - thanks iklajo

https://forum.yubico.com/viewtopic.php?f=35&t=2446

No, you simply import the public key (or use fetch if you have configured the URL on YubiKey):
gpg --import < pubkey.txt (or whatever filename you used for the public key)

Then insert the card and verify it by:
gpg --card-status

Finally trust the public key:
gpg --edit-key YOUR KEY ID
trust
choose 5 = I trust ultimately
confirm with Y
then quit

# Generate new key directly on yubikey
```bash
gpg --card-edit
admin
gpg/card> generate
Make off-card backup of encryption key? (Y/n) n

gpg: Note: keys are already stored on the card!

Replace existing keys? (y/N) yes

Please note that the factory settings of the PINs are
   PIN = '123456'     Admin PIN = '12345678'
You should change them using the command --change-pin

Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 3m
Key expires at Thu Jan 31 10:46:36 2019 CET
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Matej Lazarevic
Email address: matej.lazarevic@33barrage.com
Comment: nov
You selected this USER-ID:
    "Matej Lazarevic (nov) <matej.lazarevic@33barrage.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

quit

gpg2 --armor --export KEY_ID

```
