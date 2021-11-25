# Google Authenticator export format

> How to rename the issuers of your Google Authenticator one-time passwords

If you use Google Authenticator and scan QR codes to create new Time-based One-Time Passwords, you might have noticed that their names are sometimes in the form `Issuer (Account name)`. Google Authenticator allows to rename the `Account name` part, but not the `Issuer`.

The following explains how to rename the `Issuer`, or even get rid of it.

## Requirements

- The Google Protocol Buffers tool, which can be found [here](https://developers.google.com/protocol-buffers), or in this repository.

- The `OtpMigration.proto` file present [here](https://github.com/qistoph/otp_export/blob/master/OtpMigration.proto) — thank you to the author — or in this repository as well.

- A way to scan and create QR codes — `zbarcam` and `qrencode` for instance.

## Export, edit, and import Google Authenticator's data

In Google Authenticator settings, export your accounts. The app will display a QR code containing your TOTP data encoded with Protocol Buffers and URL encoded base64. If you have lots of accounts, then multiple QR codes will be created.

Use the next command with one QR code at a time.

I use ZSH. You might have to tweak the commands if you use a different shell.

```zsh
zbarcam | \
	sed 's/QR-Code://' | \
	sed 's/otpauth-migration:\/\/offline?data=//' | \
	sed -e 's/%2B/+/ig' -e 's/%2F/\//ig' -e 's/%3D/=/ig' | \
	base64 -d | \
	protoc --decode=MigrationPayload OtpMigration.proto \
	> secrets
```

If you retrieve the data from the QR code in another way, you can replace the first line by:

```zsh
# Note the space before `echo`, it prevents the line
# to be saved in the command history, as it contains secret data
 echo "<qr-code-data>" | \
```

Now, you can open the `secrets` file and edit it as you want. For each entry, you can rename the `issuer`, or delete it.

Then run the following to recreate a QR code and display it on your screen, which you will scan with Google Authenticator using the "Import accounts" feature present in the settings.

```sh
cat secrets | \
	protoc --encode=MigrationPayload OtpMigration.proto | \
	base64 -w 0 | \
	sed -e 's/+/%2B/ig' -e 's/\//%2F/ig' -e 's/=/%3D/ig' | \
	sed 's/^/otpauth-migration:\/\/offline?data=/' | \
	xargs echo | \
	qrencode -t utf8 -o -
```

---

Keep in mind that this data is as secret as your passwords. Do not play with QR codes in a public place where you could be seen by a security cameras, or people with phones.
