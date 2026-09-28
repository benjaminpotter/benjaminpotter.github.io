---
title: "Linux Email Setup"
date: 2026-09-28
tags: [linux, arch, vim, email]
draft: false
description: ""
---

The following is documentation (mostly for myself) about how I configure my local mail setup.
I want to sync my personal mail account hosted by Proton, to a terminal-first local interface.


## 1. Proton Mail Bridge

Proton stores all mail encrypted on the server.
In order to read it locally, we have to run `protonmail-bridge` which sits in between the Proton server and the local client.
The bridge encrypts/decrypts mail as it is passed from/to the local client.

<https://github.com/ProtonMail/proton-bridge>
<https://archlinux.org/packages/extra/x86_64/protonmail-bridge/>
<https://proton.me/mail/bridge>

### Install on Arch Linux

```
sudo pacman -S protonmail-bridge
```

The package requires a system _keychain_ to be installed.
In my case, there was no such keychain installed by Arch by default.
I liked the simple approach used by `pass`, so that is what I opted for.
Proton says that other keyrings can be used if you have something else you prefer.
See [this support article](https://proton.me/support/bridge-cannot-access-keychain) for more info.

First, we need to install `pass`.

```
sudo pacman -S pass
```

### Setup a keyring

In order to get `pass` working, we need to initialize a password store.
From the `pass` manual page:

> The init command must be run before other commands in order to initialize the password store with the correct gpg key id. 
> Passwords are encrypted using the gpg key set with init.

Initializing a password store requires an OpenPGP key created by GnuPG on Arch.
GnuPG (or `gpg` on the command-line) should already be installed as a dependency of `pacman`.
Running `gpg` for the first time on my machine yields:

```
[I] ben@thinkpad-x1 ~> gpg --list-keys
gpg: directory '/home/ben/.gnupg' created
gpg: /home/ben/.gnupg/trustdb.gpg: trustdb created
```

I think this implies I have no keys setup yet.
To [setup a new keypair](https://wiki.archlinux.org/title/GnuPG#Create_a_key_pair):

```
gpg --gen-key
```

I put in a passphrase that I generated from and saved to my Bitwarden vault.
The syntax of the `pass init` command requires we pass a "gpg-id_or_email".
It's not clear to me what the `gpg-id` or `email` fields really mean.
Are they referring to the email I used to create the key?
Does each GPG key have an id assigned to it?
The language in the documentation here could be more helpful.

The approach I took is

```
gpg --list-secret-keys --keyid-format=long
```

Which results in something like:

```
[keyboxd]
---------
sec   ed25519/XXXXXXXXXXXXXXXX 2026-09-28 [SC] [expires: 2029-09-27]
      YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY
uid                 [ultimate] Ben Potter <bp@benpot.com>
ssb   cv25519/ZZZZZZZZZZZZZZZZ 2026-09-28 [E] [expires: 2029-09-27]
```

I have replace the secret key information with X, Y, and Z.
I copied the alphanumeric id that corresponds to the Xs and used it in the `pass init` command.
I think we have to supply the gpg-id of the secret, so we use `--list-secret-keys`, but I am not sure.
The alternative is to try `--list-keys` which shows the public key id.

Anyways to setup `pass` with this GnuPG key id we do:

```
pass init XXXXXXXXXXXXXXXX 
```

<https://wiki.archlinux.org/title/Pass#Basic_usage>
<https://wiki.archlinux.org/title/GnuPG>

### Setup Proton Bridge

After `protonmail-bridge` has been installed, I run:

```
protonmail-bridge
```

We are asked for the passphrase used to setup the GnuPG key.
This is a good sign that we are on the right track.
Proton Bridge has identified that we are using `pass` and that there is a PGP key stored on that keyring.
I am not sure how this discovery works automatically and where it might fail.
YMMV here.

I signed into my Proton account and went through a few verification steps.
Once signed in, Proton Bridge provided some information about the settings I should use for my local mail client.

| Field | IMAP | SMTP |
| ----- | ---- | ---- |
| Hostname | 127.0.0.1 | 127.0.0.1 |
| Port | 1143 | 1025 |
| Username | bp@benpot.com | bp@benpot.com |
| _Password_ | XJEZ78NwZMMcNbdll2JBCA | XJEZ78NwZMMcNbdll2JBCA |
| Security | STARTTLS | STARTTLS |
| ----- | ---- | ---- |

The settings appear identical for IMAP and SMTP, except the port.
Proton Bridge serves a local IMAP and SMTP service that the local client connects to to recieve unencrypted mail that it understands.
Note that the password fields provided here _DO NOT_ correspond to the password for your Proton mail account.
They are generated locally and only apply to this connection to Proton Bridge.

We can add the local password that Proton Bridge generated to our new `pass` keyring.
Then, when we are configuring our local mail client, we don't store this password in the clear.

```
pass insert proton-bridge
```

Follow the prompts to add the password to the keyring.
Test that the password was entered correctly:

```
pass show proton-bridge
```


TODO:
Note that the bridge has to be running in order to serve the local mail server and enable connection with the local mail client.
I am not certain if Proton Bridge is configured to start on boot by default.
More on this later.

Now let's configure our chosen local mail client.


## Setup Himalaya

For configuration with Proton, the automatic setup wizard does not work.
Testing it locally, I get the following error reported from the setup wizard.
We will need to configure the account manually in the `config.toml` file.

```
[I] ben@thinkpad-x1 ~> himalaya

Welcome to Himalaya, the CLI to manage emails.

Himalaya talks to your existing mailbox over IMAP, JMAP, Gmail, Microsoft
Graph or a local Maildir. It needs one account to know which mailbox to
read, and no configuration file was found at:

  /home/ben/.config/himalaya/config.toml

The wizard discovers a provider's settings from your email address, tests
the connection and generates a ready-to-use account. Everything discovery
does not cover is written by hand, and every field is documented at:

  https://github.com/pimalaya/himalaya/blob/master/config.sample.toml

At anytime, you can create a new account with the command:

  himalaya configure

> Create a configuration with a default account? Yes
> Email: bp@benpot.com
✗ No configuration found
Error: Could not automatically discover a configuration for `bp@benpot.com`.

Write your account configuration by hand instead, starting from the documented sample:
  https://github.com/pimalaya/himalaya/blob/master/config.sample.toml

Suggestions:
 - Run with --log-level to enable more verbose logs

```


The sample configuration file can be found [here](https://github.com/pimalaya/himalaya/blob/master/config.sample.toml).
I copied the whole thing into my local configuration at `$XDG_CONFIG_HOME/himalaya/config.toml`.
This is generally what I like to do as the default configuration contains lots of useful documentation.
We can just uncomment the portions that we need.

This was where I ended up with the account section of the config.
I had to also comment out the placeholder IMAP/SMTP configurations that appear in the default config.
Otherwise, I was getting a "duplicate" error in the `imap.server` keys and the like.
It would be nice if the global keys could be overridden when the account specific configuration is applied.

```
# --------------------------------------------------------------------------------
# Account config
# --------------------------------------------------------------------------------

# One section per account, picked with `-a/--account <NAME>` or, failing that,
# by the entry flagged `default = true`.

[accounts.proton]

# Use this account when `-a/--account` is not passed.
default = true

# Address this account sends as, and the name it carries. They fill the `From`
# header of a composed message absent `--from`, and without them the header is
# left out. Also spelled `from` and `from-name`, the keys himalaya-tui writes;
# both binaries read both, one file backing the two.
email = "bp@benpot.com"
display-name = "Ben Potter"

# Signature appended after the body, and after any quoted source text, absent
# `--signature` and `--signature-file`.
#signature = "Best,\nAlice"
#signature-delim = "-- \n"

# Per-account overrides for the global options above.
#downloads-dir = "~/downloads/example"
#table.preset = "││──╞═╪╡┆    ┬┴┌┐└┘"
#table.arrangement = "dynamic"
#envelope.list.datetime-fmt = "%F %R%:z"
#envelope.list.datetime-local-tz = false

# Unique configuration for Proton mail.
# Point to the `protonmail-bridge` server that handles interop with Proton mail.

imap.server = "imap://127.0.0.1:1143"
imap.sasl.plain.username = "bp@benpot.com"
imap.sasl.plain.password.command = "pass show proton-bridge"

smtp.server = "smtp://127.0.0.1:1025"
smtp.sasl.plain.username = "bp@benpot.com"
smtp.sasl.plain.password.command = "pass show proton-bridge"
```

I also uncommented the `mailbox.alias.inbox = "INBOX"` line to set the default mailbox to use if the `-m` flag is not specified when reading mail.

TODO: Add TLS certs exported from protonmail-bridge
The button to export from the GUI is not clickable.

<https://github.com/pimalaya/himalaya>







