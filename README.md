# yubikey-guide
Simple steps on how to configure Yubico YubiKey 4 on Linux for signed git commits

## Install gpg2
__Important:__ Do not use gpg because YubiKey 4 works only with gpg2 !

## Freaquently used commands
```bash
# generate new key on computer
gpg2 --gen-key

# list secret keys in long format
gpg2 --list-secret-keys --keyid-format LONG

# list public keys in short format
gpg2 --list-keys --keyid-format SHORT

# The command(s) above will return something like this:
/home/username/.gnupg/secring.gpg
-------------------------------
sec   4096R/<COPY_LONG_KEY> 2016-08-11 [expires: 2018-08-11]
uid                          User Name <user.name@email.com>
ssb   4096R/62E5B29EEA7145E 2016-08-11

# Secret keys stored on your computer are marked with sec or ssb for subkeys, secret keys not available (for example, when
# your exported only secret subkeys running gpg --export-secret-subkeys are marked with sec#, secret key stubs only
# available on an OpenPGP smartcard (like the YubiKey also implements) with ssb>. Key stubs are simple references that a key
# is not available or is stored on a smartcard, and do not include the actual private key.

# export public key
gpg2 --armor --export PASTE_LONG_KEY_HERE > public-key.txt

# export private key
gpg2 --armor --export-secret-key PASTE_LONG_KEY_HERE > secret-key.txt
```

## Generate new key directly on yubikey (preferred method)
```bash
# enter card settings
gpg --card-edit
# request admin privileges
admin
# generate key
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

Real name: Matej
Email address: matej@xxx.com
Comment: nov
You selected this USER-ID:
    "Matej (nov) <matej@xxx.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

# exit from yubikey settings
quit
# export the generated public key
gpg2 --armor --export KEY_ID
# export the generated private key
gpg2 --armor --export-secret-key KEY_ID
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
You don't need to do this if you generated a key directly on yubikey!
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

## Import key to another computer - thanks @aussieboi

Had to download [GnuPG for OS X] https://sourceforge.net/p/gpgosx/docu/Download/


## Import key to another computer - thanks iklajo

>  ** Make sure your yubikey is not connected **

"No, you simply import the public key (or use fetch if you have configured the URL on YubiKey):
gpg --import < pubkey.txt (or whatever filename you used for the public key)"


Then insert the card and verify it by:

```
gpg --card-status
```
```
gpg --edit-key YOURKEYID
```
Finally trust the public key:
```
gpg> trust
```
```
Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y
```
choose 5 = I trust ultimately
confirm with Y
```
gpg> quit
```

## Import public key to another computer

gpg --import < pubkey.txt (or whatever filename you used for the public key)

Then insert the card and verify it by:
gpg --card-status

Finally trust the public key:
--edit-key YOUR_KEY_ID
trust
choose 5 = I trust ultimately
confirm with Y
then quit

## Yubikey Support from Git Bash [(source)](https://gist.github.com/wsargent/072319c2100ac0aea4305d6f6eeacc08)
Git Bash's gpg doesn't like GPG4Win.  If you try running GPG from inside a Git Bash shell, you get:
```
$ gpg --card-status
gpg: detected reader `Yubico Yubikey 4 OTP+CCID 0'
gpg: pcsc_transmit failed: invalid parameter (0x80100004)
gpg: apdu_send_simple(0) failed: invalid value
Please insert the card and hit return or enter 'c' to cancel: c
gpg: selecting openpgp failed: invalid argument
gpg: OpenPGP card not available: general error
```

If you use "gpg" from your window command prompt, then `gpg --card-status` will work fine, because it's coming from `C:\Program Files (x86)\GNU\GnuPG\pub`.  So you can do this from Git Bash:

```
cd "/c/Program Files (x86)/GNU/GnuPG/pub"
./gpg.exe --card-status
```

The easiest thing to do is to ensure that "/c/Program Files (x86)/GNU/GnuPG/pub" on the PATH (which it is already), and delete the "Git Bash" version.

Check from the Git Bash shell that "GnuPG" is on there:

```
echo $PATH | grep GnuPG
```

From the Administrator Command prompt, delete GPG:

```
cd "C:\Program Files\Git\usr\bin"
rm gpg.exe
```

Then close the Git Bash shell to try again.

## Debian distros issue: --card-status not working
Follow these steps: https://blog.josefsson.org/tag/scdaemon/
