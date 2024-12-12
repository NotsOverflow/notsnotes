![](images/diverse/package.svg)

# PPA like a 8055!

> Simple tutorial on how to create your own package repository for ubuntu
> allowing you to share package with the world and your computer fleet

#### What is a Personal Package Archive ?

it's a way to personalyze your ubuntu and sharing it with everybody
You can fork or edit a program, send his sources to launchpad, have it turn into a deb package easily instalable by anyone .

#### Let's start!

first you need to connect to [launchpad.net](http://launchpad.net), using your Ubuntu one account.
click on `create a new ppa` , fillup the form and you're done :D

#### Whait a minute...

ok, ok, let's setup the environement to build debian pakage :)

#### The environement

Install some required stuff:

```bash
sudo apt install gnupg pgpgpg dh-make bzr-builddeb
```

Genrate some F\*\*\*ing PGP keys:

```bash
gpg --gen-key
```

it should output something similare to this:

```bash
generator a better chance to gain enough entropy.
gpg: key XXFFDDDEEEFFDDS marked as ultimately trusted
```

use it like so:

```bash
gpg --send-keys --keyserver keyserver.ubuntu.com XXFFDDDEEEFFDDS
```

generate a ssh key:

```bash
ssh-keygen -t rsa
```

get your fingerprint :

```bash
gpg --fingerprint your@email.com
```

whitch will output something like this:

```bash
pub   rsa5241 2011-01-14 [SC] [expires: 2070-01-08]
     F741 1A0E 57EF 0A79 15A2  8F00 85FF 1547 0319 AFE5
uid           [ultimate] bob davino <best-email-ever@iam.awesome>
sub   rsa3072 2011-01-14 [E] [expires: 2070-01-08]
```

Copy your finger print (here: F741 1A0[...]AFE5) to your launchpad PGP importing section. ( ps: the 4 last bytes are your id key )
You should receive an email signed with your public pgp key.
copy the pgp message in a file.asc :

```
-----BEGIN PGP MESSAGE-----
Version: GnuPG v1

hQGMA2eTMsSWns6yAQv/ST1pOtGuubzOoNR8f+ad17IfPbvrwOTPuX2hMH8MjK0s
[ ...
... ]
5KSondbgZd2awNMjQOX5DzCauco1oQKZYdNeEEMUNQp9Z0NlYJvM58q9oaGRHeUZ
-----END PGP MESSAGE-----
```

```bash
pgp file.asc
cat file
```

Go to the link and confirm
Go to the ssh key section and paste the content of `~/.ssh/id_rsa.pub` file to import it.
finally add the fullname and email in your `~/.bashrc`

```bash
export DEBFULLNAME="Bob Dobbs"
export DEBEMAIL="subgenius@example.com"
```

```bash
. ~/.bashrc
bzr whoami "Bob Dobbs <subgenuis@example.com>"
```

#### Creating our first packet

Ok here we are going to build a really simple program, nothing complicated.
First let's create a `README` file

```bash
#just a simple program that add google repositories
```

Then a `Makefile`

```bash
SOURCEDIR=/etc/apt/sources/list.d
GOOGLEFILE=$(SOURCEDIR)/google-chrome.list

KEY1=7FAC5991
KEY2=D38B4796

all:
        echo "nothing to build"

clean:
        echo "nothing to clean"

install:
        echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > $(GOOGLEFILE)
        sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $(KEY1)
        sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $(KEY2)

uninstall:
        [ -f $(GOOGLEFILE) ] || rm $(GOOGLEFILE)
        apt-key del $(KEY1)
        apt-key del $(KEY2)
```

Actualy the `make install` will simply add google the source and corresponding keys

#### Create your first archive!

Let's create an empty folder named _google-repo_ and add a _README_ file to it.
let's archive it and build a debian package with bazaar.

```bash
tar -czvf google-repo-0.1.tar.gz google-repo
mv google-repo google-repo-bak
bzr dh-make google-repo 0.1 google-repo-0.1.tar.gz
cd google-repo
#removing stuff for emacs modules and other stuff
rm debian/*.ex debian/*.EX
rm debian/README.source debian/README.Debian
#lets commit to packaging banch
bzr add debian/source/format
bzr commit -m "Initial commit of Debian packaging."
```

lets add tow script for post installation and pre remove respectively _debian/preinst_ and _debian/prerm_.

```bash
#!/bin/bash

SOURCEDIR="/etc/apt/sources.list.d"
GOOGLEFILE="$SOURCEDIR/google-chrome.list"

KEY1="7FAC5991"
KEY2="D38B4796"

printf "Creating the sourceliste file..." \
&& echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > $GOOGLEFILE \
&& printf " Succes\n" \
|| printf " Fail\n"
printf "\nAdding key: $KEY1 -> " \
&& apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $KEY1 \
&& printf "OK" \
|| printf "ERROR"
printf "\nAdding key: $KEY2 -> " \
&& apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $KEY2 \
&& printf "OK" \
|| printf "ERROR"
```

```bash
#!/bin/bash

SOURCEDIR="/etc/apt/sources.list.d"
GOOGLEFILE="$SOURCEDIR/google-chrome.list"

KEY1="7FAC5991"
KEY2="D38B4796"

printf "Checking for $GOOGLEFILE : "
if [ -f $GOOGLEFILE ]; then
        print "FOUND"
        if rm $GOOGLEFILE ; then
                printf " and REMOVED\n"
        else
                printf " and LEFT\n"
        fi
else
        printf "CLEAR\n"
fi
printf "Removing key: $KEY1 -> " && apt-key del $KEY1
printf "Removing key: $KEY2 -> " && apt-key del $KEY2
```

Add, chmod the files and build the archive

```bash
chmod +x debian/preinst
chmod +x debian/prerm
bzr add debian/preint debian/prerm
bzr commit -m "added scripts"
```

#### Build and test the archive

```bash
# -us -uc tells it to ignore pgp stuff for now
bzr builddeb -- -us -uc
cd ..
sudo dpkg -i google-repo_0.1-1_all.deb
sudo apt update
sudo apt install google-chrome-stable
```

easy :D.
