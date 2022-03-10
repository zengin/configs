# Commit signing with YubiKey 4 on Windows

## Disclaimer
Please note that this is not a complete guide, these are only notes I took for myself from the following resources, use it at your own risk. I have tested these instructions with YubiKey 4 on Windows 11.
1. https://www.youtube.com/watch?v=fEftwheNMm8
2. https://www.youtube.com/watch?v=Y3mLBTCiccs
3. https://www.yubico.com/blog/github-now-supports-ssh-security-keys/
4. https://github.com/drduh/YubiKey-Guide
5. https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits

## Set up on a new machine

1. Install GPG4Win
```
winget install GnuPG.Gpg4win
```

You may need to restart your terminal to get GPG into your PATH.

2. Install YubiKey Manager
https://www.yubico.com/support/download/yubikey-manager/

3. Insert your YubiKey to a USB port

4. Enter card edit mode in GPG:
```
gpg --card-edit
```

5. Switch to admin mode:
```
admin
```

6. (Optional) Factory reset the key if you need to:
```
factory-reset
```

7. (Optional) Change the key's PINs:
```
passwd
```

Follow the instructions within the command to change default PINs from 123456 (non-admin) and 12345678 (admin) to your own PINs.

8. Generate a GPG key:
```
generate
```

To increase randomness, move your mouse or type on your keyboard (gpg normally shows this message but not in the `card edit` mode).

Enter your name and email as they appear in GitHub.

9. Reconnect your YubiKey by the following commands. You will need to step out of card edit mode by hitting Ctrl+C to run these commands.
```
gpg-connect-agent  killagent /bye
gpg-connect-agent  /bye
```

10. Get public key ID:
```
gpg --card-status
```

11. Copy `<id>` from the output line:
```
General key info..: pub  rsa2048/<id>
```

12. Export your public key  (replace `<id>` with the ID copied in step 11)
```
gpg -o gpg.key --armor --export <id>
```

`gpg.key` is now the file that contains your public key.

13. Upload the contents of `gpg.key` file to GitHub:
```
GitHub -> Settings -> SSH and GPG keys -> New PGP key
```

14. Set local git config (replace `<id>` with the ID copied in step 11)

This step assumes that your Gpg4Win installation location is standard (i.e. `C:\Program Files (x86)\GnuPG\bin\gpg.exe`), change that command to point to the installation location on your machine if it is not.

```
git config --global user.signingKey <id>
git config --global commit.gpgsign true
git config --global gpg.program 'C:\Program Files (x86)\GnuPG\bin\gpg.exe'
```

15. Allow touch on YubiKey for signing:
```
& 'C:\Program Files\Yubico\YubiKey Manager\ykman.exe' openpgp keys set-touch SIG ON
```

16. Create a git commit in any repository. You will be asked your PIN (non-admin PIN from step 7) and then you will need to touch the YubiKey to sign the commit. Any subsequent commits will need only the touch.

17. Confirm that the commit is signed:
```
git log --show-signature
```

18. Push your commit to GitHub and check if `Verified` badge appears.

## Using same YubiKey on multiple machines

1. Import the public key (`gpg.key` file is from step 11)
```
gpg --import gpg.key
```

2. Follow step 14.