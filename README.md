# Golang Boilerplate - GoZones

This is a boilerplate application to quickly create new Golang applications.  It can support File mode or Server mode transactions and includes additional helper functions for logging, file interactions, and other common operations - in addition to other developmental building blocks such as a Container defintion, GoReleaser configuration and matching GitHub Actions that are ready to use with no or little set up.

*This document is a work in progress and this boilerplate is likely to evolve over time*

[![Tests](https://github.com/kenmoini/golang-boilerplate/actions/workflows/test.yml/badge.svg?branch=main)](https://github.com/kenmoini/golang-boilerplate/actions/workflows/test.yml) [![release](https://github.com/kenmoini/golang-boilerplate/actions/workflows/release.yml/badge.svg?branch=main)](https://github.com/kenmoini/golang-boilerplate/actions/workflows/release.yml)

***In this documentation, "GoZones/go-zones" can be replaced with your application name***

## Example Commands & Parameters

```bash
# File Mode - input source, output target (default mode)
$ ./go-zones -source=./zones.yml -dir=./generated
# Server Mode
$ ./go-zones -mode server -config=./config.yml
```

## Deployment - As a Container

This boilerplate comes with a `Containerfile` that can be built with Docker or Podman with the following command:

```bash
# Build the container
podman build -f Containerfile -t go-zones .

# Create a config directory locally with a server configuration YAML file
mkdir config && cp config.yml.example config/config.yml

# Mount that directory and run a container
podman run -p 8080:8080 -v config/:/etc/go-zones go-zones
```

### Adding extra files to the container image

If you need additional assets along side the Golang binary in the built container you can simply place them in the `container_root` directory - directories/files in this `container_root` directory will be copied to the root of the container file system.  You can find an example of using a touchfile to create the `/etc/go-zones/` directory in the built container.

## Deployment - Building From Source

Since this is just a Golang application, as long as you have Golang v1.15+ then the following commands will do the job:

```bash
go build

./go-zones
```

Of course, once you change the name of the application the executable name will change as well.

## Starting Development

### Initial Changes

You'll likely want to change a few things, namely the name.

- You can find the `appName` defined in the `variables.go` file.  This is what the application references internally for logs and so on.
- You will also need to change the name/path in the `go.mod` file to match your repository path/name.  This is where the executable package gets its name.
- Rename the template touch folder in `container_root/etc/go-zones/`.
- Change the references in this `README.md` file to match your application name.
- Adjust the `.gitignore` file to match what will be the name of your executable package.
- Modify the `go-zones.service` file to match if you're utilizing this in Server Mode with SystemD as a Service

## Lifecycle

As the versioning of your application progresses, make sure to keep that semantic version up to date in the `appVersion` variable defined in the `variables.go` file.

Once you are ready to release a new version of your application, you can utilize GoReleaser and GitHub Actions to create packaged Go binaries of your application across a matrix of operating systems and architectures.

## Creating a Release

In order for GoReleaser to create release assets, it needs a GPG Signing Key.  You could create an individual signing key, or use your own personal key, however it's suggested to use a tiered key chain to allow easier management and revocation.

### Generating a Personal GPG Key

If you already have a Personal GPG Key pair that provides your identity then you can skip this step.

```bash
## Generate a Personal Signing GPG Key
gpg --full-generate-key
```

- Select option `(1) RSA and RSA (default)`
- Enter `4096` for the keysize 
- Set `0` for no expiration (unless you really wanna mess with renewing your personal key...)
- Confirm the settings are correct with `y`
- Enter your personal information in the following prompts
- Provide a passphrase to encrypt your personal key

Now you have a key in the key store - you can verify this by running the command `gpg --list-keys`

### Generating Project GPG Keys

Next we'll make a key for the project.

```bash
gpg --expert --full-generate-key

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
  (14) Existing key from card
Your selection? 8
```

The next set of prompts will allow you to toggle capabilities - toggle signing and encrypting until the only allowed action is "Certify"

```bash
Possible actions for a RSA key: Sign Certify Encrypt Authenticate 
Current allowed actions: Sign Certify Encrypt 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s

Possible actions for a RSA key: Sign Certify Encrypt Authenticate 
Current allowed actions: Certify Encrypt 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? e

Possible actions for a RSA key: Sign Certify Encrypt Authenticate 
Current allowed actions: Certify 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q
```

- Set the key size to `4096`
- Set an expiration (or don't)
- For the **"Real Name"** field set it for your Project name, probably the name of your repo - set the Email to whatever valid address you'd like
- In the **"Comment"** field set an overall reference to your project such as `github.com/YOUR_USER/PROJECT_NAME Project Key`
- Give this key a passphrase, but a different one than your personal key of course.

This project key with the Certify capability can now sign other keys, like a Release Signing Key.

Before proceeding, you should see similar output to the following:

```
You selected this USER-ID:
    "golang-boilerplate (github.com/kenmoini/golang-boilerplate Project Key) <ken@kenmoini.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key A34134B8D2EC5887 marked as ultimately trusted
gpg: revocation certificate stored as '/Users/kenmoini/.gnupg/openpgp-revocs.d/AB8491501376C56B72BDDD24A34134B8D2EC5887.rev'
public and secret key created and signed.

pub   rsa4096 2021-03-06 [C]
      AB8491501376C56B72BDDD24A34134B8D2EC5887
uid                      golang-boilerplate (github.com/kenmoini/golang-boilerplate Project Key) <ken@kenmoini.com>
```

Take a note of the Key ID, `AB8491501376C56B72BDDD24A34134B8D2EC5887` in this case, printed right above the `uid` line.

Use the following command to save that Key ID to an environment variable: `export PROJECT_KEY_ID=AB8491501376C56B72BDDD24A34134B8D2EC5887`

### Generating Project Release Signing GPG Keys

Now that we have a Project Key, we'll create a Release Signing Key - this allows your Personal and Project Keys to stay offline safely, while you use a Release Signing Key in GitHub to sign the project.  If GitHub's secret store is compromised then you can easily revoke the Release Signing Key and generate a new one without compromising your more sensitive Personal and Project Keys.

Add a new key to your Project Key with the following command:

```bash
gpg --expert --edit-key $PROJECT_KEY_ID

gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
sec  rsa4096/A34134B8D2EC5887
     created: 2021-03-06  expires: never       usage: C   
     trust: ultimate      validity: ultimate
[ultimate] (1). golang-boilerplate (github.com/kenmoini/golang-boilerplate Project Key) <ken@kenmoini.com>

gpg>
```

- This opens the `gpg>` terminal prompt - type in `addkey` and select option `(4) RSA (sign only)`
- Set the key size to `4096`
- Set an expiration if you'd like
- Confirm and then DOUBLE confirm
- You'll be prompted for the passphrase to the Project Key so it can be used to sign this new key
- Finalize and exit the `gpg>` prompt by typing in `save`

Now that the key is created you can sign it with your root Personal Key by running the following:

```bash
gpg --sign-key $PROJECT_KEY_ID
```

Which should result in something similar:

```bash
gpg --sign-key $PROJECT_KEY_ID

sec  rsa4096/A34134B8D2EC5887
     created: 2021-03-06  expires: never       usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa4096/95ED8AE57849D064
     created: 2021-03-06  expires: never       usage: S   
[ultimate] (1). golang-boilerplate (github.com/kenmoini/golang-boilerplate Project Key) <ken@kenmoini.com>


sec  rsa4096/A34134B8D2EC5887
     created: 2021-03-06  expires: never       usage: C   
     trust: ultimate      validity: ultimate
 Primary key fingerprint: AB84 9150 1376 C56B 72BD  DD24 A341 34B8 D2EC 5887

     golang-boilerplate (github.com/kenmoini/golang-boilerplate Project Key) <ken@kenmoini.com>

Are you sure that you want to sign this key with your
key "Ken Moini (Kemo Personal Signing Key) <ken@kenmoini.com>" (ABF6891102897F05)

Really sign? (y/N) y
```

Confirm the prompt and enter your Personal Key passphrase when prompted.

### Exporting Keys

Now that the keys are all created and signed, let's export the keys to a file, and to the public key network.

```bash
gpg --keyid-format long -K $PROJECT_KEY_ID
```

You should see output similar to the following, look for the `[S]` signing key:

```
gpg --keyid-format long -K $PROJECT_KEY_ID

gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
sec   rsa4096/A34134B8D2EC5887 2021-03-06 [C]
      AB8491501376C56B72BDDD24A34134B8D2EC5887
uid                 [ultimate] golang-boilerplate (github.com/kenmoini/golang-boilerplate Project Key) <ken@kenmoini.com>
ssb   rsa4096/95ED8AE57849D064 2021-03-06 [S]
```

Set the signing key ID to an environment variable like so:

```
export SIGNING_KEY=95ED8AE57849D064
```

Finally, export it to an ASCII file:

```bash
gpg --armor --export-secret-subkeys $SIGNING_KEY\! > golang-boilerplate.signing-key.gpg
```

You'll be prompted for the Project Key passphrase - enter your passphrase.  Ensure to protect this private key file on your file system.

Export your Project Key to the public key network (optional)

```bash
gpg --send-key $PROJECT_KEY_ID
```

### Creating the Repository Secrets in GitHub

Now that we have our full GPG Key Chain set up, we can now create the GitHub Repository Secrets needed by GoReleaser.

1. Navigate to your GitHub Repository
2. Click on the ***Settings*** tab
3. Click the ***Secrets*** link from the left pane
4. Click the ***New repository secret*** button
5. Create a new Repository Secret

  - **Name**: GPG_PRIVATE_KEY
    **Value**: The contents of your exported signing key file

### Creating a New Release in GitHub

With the Repository Secret in place, you can now create a new Release.

1. Navigate to your GitHub Repository
2. In the right-hand pane, under ***Releases*** click the *Create a new release* link
3. Give it a tag in the versioned semantic format like such: `v0.0.1`
4. Fill in the Title and Description, click Create

Now you can navigate to the ***Actions*** tab and watch the Release pipeline run - once the pipeline completes you'll have additional compiled assets in that release.
