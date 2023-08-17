---
Title: Qhat you need to know about using SSH with Github and Gitlab
date: 2023-08-14 2:58PM
categories: [Git]
tags: [github, gitlab, ssh]
---
## Introduction
SSH uses an asymmetric key cryptography system. These keys are used to authenticate, ie. know who you are by the Gitlab or Github servers. Essentially, these are your replacement to providing username and password when pushing and cloning/pulling from Github/Gitlab. Although, you can set up a password in your private keys when creating one. That password you created during key creation is used to keep your private key in your computer secure by the means of encryption and is never sent to Gitlab or Github, yes, you are using symmetric encryption to protect your asymmetric keys. **If that private key along with its password is stolen or lost, people can do whatever they want in your gitlab or github account in your name. So you want to be extra careful to make sure your private key is secured.**

You need to understand how digital signatures work in order to fully understand what is going on here. But in summary, your computer signs a small bit of data using the private key on your computer and send to the server. The server then uses your public key to verify that signature and know it is indeed you. **Your private key should be treated like your password. Do not give it to anyone else. Noone other than you need it. Not even Github and Gitlab need them to validate. It is fine to provide your public key though and that is what the other party uses to verify or encrypt you a message.**

I am going to share two methods of using SSH to Log in Github and Gitlab. Because Github and Gitlab work the same way including how to setup I will just use Gitlab as an example. I am unsure about other Git service providers such as Gittea or BitBucket so you will have to dig out information about them yourself.

## Prerequisits
### For Windows users
* Make sure that GitBash is installed
* Your ssh folder could be located in ``C://Users/<your_username>/.ssh/``. You may want to enable ``Show Hidden Folders`` option if you do not see it
* Windows uses backslash(\\) to indicate directory path, however, you are supposed to use forwardslash(/) in the config file
* Tick ``File name extensions`` under File explorer View tab. You should always keep it enabled

#### Remove all the user permission other than the owner
If you get an error ``Bad owner or permissions on C:\\Users\user\.ssh\config`` after you follow the instructions during any further ssh connections, try to remove all the user permissions and inherited permissions from ``.ssh`` folder. Follow the following steps to achive that
* Right click on ``.ssh`` folder and go to Properties
* Go to ``Security`` tab
* Click on ``Advanced``
* Click on ``Disable inheritance`` in the bottom left screen
* Select ``Remove all inherited permissions from this object`` if asked
* You may be given a warning stating only the owner can change permission and all other will be lost, select ok

### For GNU/Linux users
* You will have to install ``git`` package following your distribution's package manager, most have them in their official repository
* Your ssh folder is located under ``~/.ssh/`` which is a hidden directory starting with a ``.`` so you may want to use ``ls -a`` to see it in your home directory

## Generate SSH keys
The first step to using SSH keys is to have one. Keep in mind where you save them. It is better to just save them in your ssh folder which is the default option. If you want to use multiple accounts, you need one for each account and you have to give them different names. For single account, it is fine to just keep pressing enter without giving much thought. You can either use ``ed25519`` or ``rsa`` key. Gitlab recommends you to use ``ed25519`` key.

* For ``ED25519`` key:
```bash
ssh-keygen -t ed25519 -C "<comment>"
```
Where comment can be anything and it cannot be left blank.

* For ``RSA 2048-bit`` key:
```bash
ssh-keygen -t rsa -b 2048 -C "<comment>"
```
Where comment can be anything and it cannot be left blank.
You will see something like this output
```
Enter file in which to save the key (C:\Users\user/.ssh/id_ed25519): personal_key
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in personal_key.
Your public key has been saved in personal_key.pub.
The key fingerprint is:
SHA256:4lvpSRHpwZ1WCyj0cMESluoDiRJvpRQTSQcsbCJFWbM mykeyComment
The key's randomart image is:
+--[ED25519 256]--+
|.=OB+.=+oo. .    |
|=o=o.++=oo + .   |
|+* +E. o* + .    |
|o * .  . +       |
|.. o  . S        |
|    o. . o       |
|     .. +        |
|       = .       |
|      . o        |
+----[SHA256]-----+
```

* In line 1, you can provide full path to where you want to save your keys. If you do not provide anything, the one in the parenthesis is going to be used. If you just provide the file name, your keys will be saved in your current working directory. Your current working directory being where you were when you run that command.

* In line 2 and 3, you will be asked for password, you can enter your password and press on Enter. But you dont have to. If you leave it empty and just press enter, no password will be set.

* In line 4, it says ``Your identification has been saved in personal_key`` where ``personal_key`` is your private key. You should keep it private.

* In line 5, it says ``Your public key has been saved in personal_key.pub`` that key with ``.pub`` extension is your public key.

* In line 7, that string ``4lvpSRHpwZ1WCyj0cMESluoDiRJvpRQTSQcsbCJFWbM`` is your key fingerprint. There are many keys in Gitlab and Github servers. You can create as many keys as you want but there will only ever be one key with that exact fingerprint matching letter by letter and that is your key. Because that key has a unique fingerprint and you will tie that key with your account, gitlab knows that key is yours. Whenever someone submits that key, gitlab will know it is you.

You are free to decide whether you want to use a password or not. This password is only used locally to encrypt and protect your private key and will not be sent to your gitlab in any way. You will be prompted to type in this password when pushing or cloning. Regardless of whether you decide to use password or not, make sure to keep this key safe and not allow others to have it. You may have issues with some alternative git clients that are not written with password protected keys in mind but it will have no issues when using with command line.

You will do the same for as many accounts you want to make

In my case, my public key generated above for ``personal_key.pub`` looks like
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDvsX4vRhNZYrBPQTsjXVfys8kvyRoTPTuvSauOF4hiA mykeyComment
```
**This is the key you want to give to others including Gitlab**

## Scenario 1: Using different SSH keys to authenticate with different accounts in Gitlab
As an example, say you have two different accounts, one for your personal use and one for your professional use. Say that your personal user gitlab username is ``personal_gitlab`` and your professional user gitlab username is ``professional_gitlab``

### Steps

#### 1: Copy your public key contents and put it in gitlab

**You can either do it in user settings or in Repository settings. You usually want to do it for an entire user profile**

You want to copy all the contents from your public key by opening it in a text editor such as notepad. Then,
* Login to Gitlab or Github
* Click on your user profile and click on Settings
* Select SSH keys [in github it could be under GPG and SSH keys]
* Copy the public key exactly, in my case I would copy the above from ``ssh-ed25519`` to ``mykeyComment`` including both
* Your key title or comment may be automatically set to ``mykeyComment`` which is fine
* You may want to set expiry date but it is generally not necessary
* Save it

You will paste ``personal_key.pub`` to your personal gitlab account and ``professional_key.pub`` to your professional gitlab account

#### 2: Create config file for ssh
Now you need to create a configuration file using a text editor such as notepad for ssh to use. Keep in mind that this has to be saved in your ssh folder with the name ``config`` and no extension. **Make sure there is no file extension like txt and that it is in your ssh folder. Also use simple text editor such as Notepad and not a Word processor like Microsoft Word**

This is what the config looks like
```
Host nick1
   IdentityFile C://Users/user/.ssh/personal_key
   User git
   Port 22
   PreferredAuthentications publickey
   HostName gitlab.com

Host nick2
   IdentityFile C://Users/user/.ssh/professional_key
   User git
   Port 22
   PreferredAuthentications publickey
   HostName gitlab.com

Host *
   IdentifyFile C://Users/user/.ssh/default_key
   User git
   Port 22
   PreferredAuthentications publickey
   HostName gitlab.com
```

To clarify, none of the enteries in that config file contain your gitlab or github username. Your actual github and gitlab usernames come when creating a ``remote`` for your repositories. The indentation is not necessary per say but will make it easier to read for other people.

* ``nick1`` and ``nick2`` are names that you give for your keys. Rather than calling them ``personal_key`` and ``professional_key`` you call them ``nick1`` and ``nick2`` from now on.

* ``IdentifyFile`` is the path to your private key file. Make sure to put in full path name. In GNU/Linux or UNIX systems you can use the ``~`` character to refer to your home directory.

* ``User`` is the UNIX username of the git server. **It is not your gitlab or github username as you may think** Unless you have a selfhosted gitlab server, you are always going to set it to ``git`` user. And it is the same for Gitlab and Github as many things are.

* ``Port`` defines the port number ssh is listening on the server, by default the port is ``22`` and Github and Gitlab both use the default port. Change it only in case you need to.

* ``PreferredAuthentications`` defines what kind of Authentication mechanism do you wish to use with the server. Gitlab and Github only accept ``publickey`` as an authentication mechanism. If your ssh server required the use of password, you would set it to ``password`` instead but it is not the case here. If you absolutely want to force using public key in case of weird issues, you may also set ``PubkeyAuthentication`` to ``yes`` or ``no`` in a similar way.

* ``HostName`` is where you put in the URL of your git server. If it is gitlab, it will be ``gitlab.com`` and if its github, it will be ``github.com`` without any ``https://``. In case of selfhosted system, you would put in the hostname of your selfhosted server including any subdomains.

* ``Host *`` matches to all the host. This is a default fallback entry. If you need to have a default identity, you put it here. It is not necessary to have but I have mentioned it nontheless

**It is possible for you to use one key for Gitlab, another key for Github and third key for something else. Just change ``Host`` accordingly**

#### 3: Creating remote for your repository
In this new paradigm, you will have to create a local git repository and push it to gitlab by changing its origin URL. If you are new, follow these steps exactly and wont have any confusion, although if you are more understanding, you may know quicker ways to do it. This is written for everybody.

##### 1: Go to Github or Gitlab and create a new repository
* Create a new repository. **Initialize without a readme**

Suppose, the repository is named ``myrepo``

* Copy the ``ssh`` clone url for that repository

Suppose, the git clone URL in ssh is as follows
```
git@gitlab.com:personal_gitlab/myrepo.git
```

##### 2: Create a local git repository
* Create a new repository using the command
```bash
git init
```
* After your new repo is initialized, set it up with local username and email
```bash
git config --local user.name "MyNameForThisRepo"
git config --local user.email "myemailforthisrepo@example.com"
```

**You probably want to have two different name and email for two different repo. You will do it for each repo but make sure that email is valid for that github or gitlab account**

##### 3: Change the origin in that repository
Remember the ``config`` file above? I said that we will use the name ``nick1`` for our personal repo and ``nick2`` for our professional repo? What information is stored under that name ``nick1`` ?
* Private key to use
* UNIX user of the server
* Hostname of the server

Which means we can just use the name ``nick1`` and infer to the above information. The only information we did not store was our username for the gitlab. So what that means is, our origin link can be as short as
```
nick1:personal_gitlab/myrepo.git
```
Here, the first one is the alias, the one after colon(:) is your gitlab/github username and the last part is the name of the repo. It could also be a group name in place of username because Gitlab will know you have access to write in there based on your public key. So all that remains is to set it as the origin URL. To do this, do one of the following

* If it is a brand new repo and origin link wasnt previously set
```bash
git remote add origin nick1:personal_gitlab/myrepo.git
```

* If you have previously set origin and now wish to change
```bash
git remote set-url origin nick1:personal_gitlab/myrepo.git
```

You would repeat this twice, one for personal gitlab, another for professional gitlab

Now you can add, commit and push as usual
```bash
git add file1
git commit -m "commit message"
git push -u origin main
```

## Scenario 2: Using single SSH key to authenticate with a single account
Even though it is quick and easy, it takes away freedom and potentially cause more headache once you want to have multiple accounts on the same server. This is the reason I put it second. You can however have one account in Github and one account in Gitlab with the same public key. As an example, your gitlab username is ``myuser`` Unlike in previous step,
* You dont have to create config file, matter of fact, you dont even have to have it in your ssh folder
* You dont have to change ``origin`` url, just use what is given to you at the time of repo creation

### Steps
#### 1: Copy your public key contents and put it in gitlab

**You can either do it in user settings or in Repository settings. You usually want to do it for an entire user profile**

You want to copy all the contents from your public key by opening it in a text editor such as notepad. Then,
* Login to Gitlab or Github
* Click on your user profile and click on Settings
* Select SSH keys [in github it could be under GPG and SSH keys]
* Copy the public key exactly, in my case I would copy the above from ``ssh-ed25519`` to ``mykeyComment`` including both
* Your key title or comment may be automatically set to ``mykeyComment`` which is fine
* You may want to set expiry date but it is generally not necessary
* Save it

#### 2: Create a new repository on Gitlab
In this case, you only will have one gitlab or github account
##### 1: Go to Github or Gitlab and create a new repository
* Create a new repository. **Initialize without a readme**

Suppose, the repository is named ``myrepo``

* Copy the ``ssh`` clone url for that repository

Suppose, the git clone URL in ssh is as follows
```
git@gitlab.com:myuser/myrepo.git
```

##### 2: Clone that repository on your computer
* Clone using git clone
```bash
git clone git@gitlab.com:myuser/myrepo.git
```
* After your new repo is cloned, set it up with global username and email since you will only have this one account
```bash
git config --global user.name "MyName"
git config --global user.email "myemailo@example.com"
```
**You only need to set global username and email once also make sure it is as provided in gitlab/github**

##### 3: Add your content and add, commit and push
* Add your content, whatever you wanted to put in that repo
* Add, commit and push that to gitlab

```bash
git add file1
git commit -m "commit message"
git push -u origin main
```

> I will admit this was quiet a long post. But I hope all of your confusion is gone now regarding this
