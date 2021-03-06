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

1. Select option `(1) RSA and RSA (default)`
2. Enter `4096` for the keysize 
3. Set `0` for no expiration (unless you really wanna mess with renewing your personal key...)
4. Confirm the settings are correct with `y`
5. Enter your personal information in the following prompts
6. Provide a passphrase to encrypt your personal key

Now you have a key in the key store - you can verify this by running the command `gpg --list-keys --keyid-format long`

You could stop here at this step and use your personal key to sign application releases but that risks everything else that would be signed by your personal key if it were ever compromised.  To avoid this and better manage GPG keys, we'll create a Project Key and sign it with our Personal Key.

### Generating Project GPG Keys

Next we'll make a key for the project.  It's pretty much the same process as the last, but make sure to change the Name and Email address of the Project Key.

```bash
## Generate a PROJECT GPG Key
gpg --full-generate-key
```

1. Select option `(1) RSA and RSA (default)`
2. Enter `4096` for the keysize 
3. Set `0` for no expiration (unless you really wanna mess with renewing your project key...)
4. Confirm the settings are correct with `y`
5. For the **"Real Name"** field set it for your Project name, probably the name of your repo - set the Email to a `project+your@email.com` format
6. In the **"Comment"** field set an overall reference to your project such as `github.com/YOUR_USER/PROJECT_NAME Project Key`
7. Give this key a passphrase, but a different one than your personal key of course.

You should see similar output to the following if everything was entered and formatted correctly:

```
GnuPG needs to construct a user ID to identify your key.

Real name: golang-boilerplate
Email address: golang-boilerplate+ken@kenmoini.com
Comment: github.com/kenmoini/golang-boilerplate Project Key
You selected this USER-ID:
    "golang-boilerplate (github.com/kenmoini/golang-boilerplate Project Key) <golang-boilerplate+ken@kenmoini.com>"
```

Before proceeding, we need to set an environmental variable that defines our Project Key ID - you can get it by running the following command:

```bash
gpg --list-keys --keyid-format long
```

You'll see something similar to the following:

```
$ gpg --list-keys --keyid-format long

gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
/home/kemo/.gnupg/pubring.kbx
-----------------------------
pub   rsa4096/FC9869CA98201DA9 2021-03-06 [SC]
      4FDB20A73BA4D3AC557C56F9FC9869CA98201DA9
uid                 [ultimate] Ken Moini (Kemo Personal Key) <ken@kenmoini.com>
sub   rsa4096/C2C7C3AB6EB04E32 2021-03-06 [E]

pub   rsa4096/071BC18D1BB25158 2021-03-06 [SC]
      E395F97DDE8521223AB5F704071BC18D1BB25158
uid                 [ultimate] golang-boilerplate (github.com/kenmoini/golang-boilerplate Project Key) <golang-boilerplate+ken@kenmoini.com>
sub   rsa4096/C6761B6899E6FB04 2021-03-06 [E]
```

What you're looking for is the long key ID for your Project key in between the `pub` and `uid` lines - in this case it would be `E395F97DDE8521223AB5F704071BC18D1BB25158`.

Save that Project Key ID to an environment variable with the following command, of course substituting with your Key ID:

```bash
export PROJECT_KEY_ID=E395F97DDE8521223AB5F704071BC18D1BB25158
```

### Generating Project Release Signing GPG Keys

Now that we have a Project Key, we'll create a Release Signing Key - add a new key to your Project Key with the following command:

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

1. This opens the `gpg>` terminal prompt - type in `addkey` and select option `(8) RSA (set your own capabilities)`
2. Enter `Q` for finished which should use the default actions of "Sign Encrypt"
3. Set the key size to `4096`
4. Set an expiration if you'd like
5. Confirm and then DOUBLE confirm
6. You'll be prompted for the passphrase to the Project Key so it can be used to sign this new key
7. Finalize and exit the `gpg>` prompt by typing in `save`

### Signing GPG Keys

Now that we have a Project Key, we'll sign it with our Personal Key - this allows your Personal Key to stay offline safely, while you use a Project Key in GitHub to sign the project.  If GitHub's secret store is compromised then you can easily revoke the Project Key and generate a new one without compromising your more sensitive Personal Key.

***NOTE:*** *You could also create a subkey in the project key specifically for signing - that is outside the scope of this README*

Sign your Project Key with your Personal Key with the following command:

```bash
gpg --sign-key $PROJECT_KEY_ID
```

Which should result in something similar:

```bash
gpg --sign-key $PROJECT_KEY_ID

sec  rsa4096/071BC18D1BB25158
     created: 2021-03-06  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/C6761B6899E6FB04
     created: 2021-03-06  expires: never       usage: E
[ultimate] (1). golang-boilerplate (github.com/kenmoini/golang-boilerplate Project Key) <golang-boilerplate+ken@kenmoini.com>


sec  rsa4096/071BC18D1BB25158
     created: 2021-03-06  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
 Primary key fingerprint: E395 F97D DE85 2122 3AB5  F704 071B C18D 1BB2 5158

     golang-boilerplate (github.com/kenmoini/golang-boilerplate Project Key) <golang-boilerplate+ken@kenmoini.com>

Are you sure that you want to sign this key with your
key "Ken Moini (Kemo Personal Key) <ken@kenmoini.com>" (FC9869CA98201DA9)

Really sign? (y/N) y
```

Confirm the prompt and enter your Personal Key passphrase when prompted.

### Exporting Keys

Now that the keys are all created and signed, let's export the key to a file, and to the public key network.

```bash
gpg --armor --export-secret-keys $PROJECT_KEY_ID > golang-boilerplate.project-key.gpg
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
5. Create a new set of Repository Secrets

  - **Name**: GPG_PRIVATE_KEY
    **Value**: The contents of your exported Project Key file

  - **Name**: PASSPHRASE
    **Value**: The passphrase to your Project Key

### Creating a New Release in GitHub

With the Repository Secret in place, you can now create a new Release.

1. Navigate to your GitHub Repository
2. In the right-hand pane, under ***Releases*** click the *Create a new release* link
3. Give it a tag in the versioned semantic format like such: `v0.0.1`
4. Fill in the Title and Description, click Create

Now you can navigate to the ***Actions*** tab and watch the Release pipeline run - once the pipeline completes you'll have additional compiled assets in that release.
