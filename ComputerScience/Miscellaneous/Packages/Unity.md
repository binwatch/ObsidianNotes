
## Install

I tried to install Unity Hub on my Ubuntu22.04 according to the [docs](https://docs.unity3d.com/hub/manual/InstallHub.html?_ga=2.217848208.703771928.1668153040-1068536422.1668153040#install-hub-linux), but have encountered 2 problems.

````ad-help
title: [1] apt-key deprecation error

**\[Problem\]** ❓ I got warning about when running:

```shell
wget -qO - https://hub.unity3d.com/linux/keys/public | sudo apt-key add -
```

That's because `apt-key` is deprecated. To see details, use `man apt-key 8` and check DEPRECATION section

**[Solution]** ✅ Add repo instead:

```shell
sudo sh -c 'echo "deb https://hub.unity3d.com/linux/repos/deb stable main" > /etc/apt/sources.list.d/unityhub.list'
```
````

I have also met `chmod: cannot access '/opt/unityhub/chrome-sandbox': No such file or directory` error message when running `sudo apt-get install unityhub`, but accroding to @[arsh5620](https://answers.unity.com/users/1264339/arsh5620.html)'s answer to [the same question](https://answers.unity.com/questions/1906602/error-chmod-cannot-access-optunityhubchrome-sandbo.html) it can be ignored.

````ad-help
title: [2] openssl version problem

**\[Problem\]** ❓ After installation I started Unity Hub and found it keep loading. [Running Unity on Ubuntu 22.04](https://forum.unity.com/threads/running-unity-on-ubuntu-22-04.1284083/) mentions a Unity Editor Known Issues: 

```ad-quote
Ubuntu 22.04 now ships with libssl3 by default and does not include libssl1.1. Unity currently uses .NET5 which requires libssl1.1. To work around this issue you can install libssl1.1 from an older Ubuntu release
```

**[Solution]** ✅ However, the solution given did not work for me, then I found some other solutions in [this question](https://askubuntu.com/questions/1403619/mongodb-install-fails-on-ubuntu-22-04-depends-on-libssl1-1-but-it-is-not-insta):

Force the installation of libssl1.1 by adding the ubuntu20.04 source, then delete the list file (by @[Lionep](https://askubuntu.com/users/414203/lionep) and [Damian](https://askubuntu.com/users/632086/damian)):

```shell
echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list

sudo apt-get update
sudo apt-get install libssl1.1

sudo rm /etc/apt/sources.list.d/focal-security.list
```
````
