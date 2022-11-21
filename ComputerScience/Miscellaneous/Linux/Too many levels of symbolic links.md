When I try to install Zotero on my Ubuntu 22.04 according to [this guide](https://www.ubuntubuzz.com/2020/06/zotero-focal-fossa.html), I met the "too many levels of symbolic links" with failure to open Zotero after creating a symbolic link to `zotero.desktop` using this command:

```shell
ln -s zotero.desktop ~/.local/share/applications/zotero.desktop
```

Then I found [a same problem](https://unix.stackexchange.com/questions/141436/too-many-levels-of-symbolic-links) as mine on StackExchange, thanks to @[Steen Schütt](https://unix.stackexchange.com/users/28919/steen-sch%c3%bctt)'s answer:

````ad-quote
As Dubu points out in a comment, the issue lies in your **relative paths**. I had a similar problem symlinking my nginx configuration from `/usr/local/etc/nginx` to `/etc/nginx`. If you create your symlink like this:

```shell
cd /usr/local/etc
ln -s nginx/ /etc/nginx
```

You will in fact make the link /etc/nginx -> /etc/nginx, because the source path is relative to the link's path. The solution is as simple as using absolute paths:

```shell
ln -s /usr/local/etc/nginx /etc/nginx
```

If you want to use relative paths and have them behave the way you probably expect them to, you can use the `$PWD` variable to easily add in the path to the current working directory path, like so:

```shell
cd /usr/local/etc
ln -s "$PWD/nginx/" /etc/nginx
```

Make sure that the path is in double quotes, to make sure things like spaces in your current path are escaped. Note that you must use double quotes when doing this, as `$PWD` will not be substituted if you use single quotes.
````

So I changed the command to solve my problem:

```shell
ln -s "$PWD/zotero.desktop" ~/.local/share/applications/zotero.desktop
```