---
author: Radi Radev
pubDatetime: 2023-05-17T17:18:00Z
title: LXPLUS Passwordless Kerberos for VSCode (+ Windows)
postSlug: lxplus-passwordless-vscode
featured: true
draft: false
tags:
  - kerberos
ogImage: ""
description:
  How to setup passwordless kerberos authentication for VSCode on the LXPLUS service at CERN and how to make it work on Windows.
---

## Introduction

The Visual Studio Code Remote SSH [extension](https://code.visualstudio.com/docs/remote/ssh) provides a very convenient way of working on a remote machine, enabling us to use all the benefits of an IDE, while in a remote environment. 

The LXPLUS service at CERN is a way for users to access compute and filesystems available within the CERN network. In this short post I show how to setup an `ssh` config to enable us to seamlessly connect to LXPLUS and VMs within the CERN network.



## The `ssh` config
On a Linux or Mac system we can edit the `ssh` config located in `/home/username/.ssh/config` (on Windows we can use [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) to do the same). Insert the following into the file by replacing `username` with your CERN username and `vm` with the host that we are trying to connect to. This configuration allows us to setup a jump connection using the `lxtunnel` host if we are not on the CERN network, for example if we are working remotely.

```
Host vm
  HostName vm.cern.ch

Host *.cern.ch
    # your CERN username
    User username
    GSSAPITrustDns yes
    GSSAPIAuthentication yes
    GSSAPIDelegateCredentials yes

Match host *.cern.ch,!lxtunnel*.cern.ch,!lxplus*.cern.ch !exec "hostname -A | grep -q '.cern.ch '"
  ProxyJump lxtunnel.cern.ch 
```

Now using the terminal we can get a kerberos ticket by the command: 

```
kinit username@CERN.CH
```

And that should be it (if we are working on a Linux system)! Now we can connect from VSCode or using the terminal with:

```
ssh username@vm.cern.ch 
```

## The tricky part - Windows

Since there is no official support from the Remote SSH extension to use WSL, by default it uses the Windows ssh client. There is however hacky way of using WSL to make the `ssh` connection instead. We need to create a `.bat` file with the following content. 

```
C:\Windows\system32\wsl.exe ssh %*
```

To point Remote-SSH extension to use that `.bat` file, we begin by opening the commmand pallete using `ctrl + shift + p` and selecting `> Remote-SSH: Settings`. From there we find the `Remote.SSH: Path` setting and add the absolute path of the `.bat` file. 

To make this work seamlessly, we must make sure the contents of the ssh config files are the same in WSL and the default one in Windows. In Windows that file is usually located in `C:\Users\username\.ssh\config` and in WSL it is usually located in `/home/rradev/.ssh/config`. After this, this configuration should let you connect to lxplus or any virtual machines behind the CERN firewall.

This process could be further automated by using the Windows equivalent of `symlink` to make sure that the changes in one of the files is always reflected in the other.