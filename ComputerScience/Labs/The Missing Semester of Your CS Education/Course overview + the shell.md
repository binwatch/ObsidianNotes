
## Exercises

### 1-4

1. `echo $SHELL`
2. `mkdir /tmp/missing`
3. `man touch`
4. `touch /tmp/missing/semester`

### 5

```shell
echo '#!/bin/sh' >> /tmp/missing/semester
echo 'curl --head --silent https://missing.csail.mit.edu' >> /tmp/missing/semester
```

When I try to use `echo "#!/bin/sh" >> /tmp/missing/semester`, something strange occured, after I typed` echo "#!/bin/sh"` then I press the space key and found I cannot enter any more character with the following prompt:

> bash: !/bin/bash: event not found

From the Bash [quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html) manual page:

| Quoting                                                                                      | Utility                                                            |
| -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| [Escape Character](https://www.gnu.org/software/bash/manual/html_node/Escape-Character.html) | How to remove the special meaning from a single character. |
| [**Single Quotes**](https://www.gnu.org/software/bash/manual/html_node/Single-Quotes.html)   | How to inhibit **all** interpretation of a sequence of characters. |
| [**Double Quotes**](https://www.gnu.org/software/bash/manual/html_node/Double-Quotes.html) | How to suppress **most** of the interpretation of a sequence of characters.                                                               |                                                                    |                                                                                             |                                                                    |
| [ANSI-C Quoting](https://www.gnu.org/software/bash/manual/html_node/ANSI_002dC-Quoting.html)| How to expand ANSI-C sequences in quoted strings. |
| [Locale Translation](https://www.gnu.org/software/bash/manual/html_node/Locale-Translation.html) | How to translate strings into different languages. |

命令行下，双引号里面用 !，Shell 会以为要执行历史展开，从而可能导致上面的报错（或者执行历史展开）。

### 6

```shell
/tmp/missing/semester
```

> bash: /tmp/missing/semester: Permission denied

From [ArchWiki](https://wiki.archlinux.org/title/File_permissions_and_attributes), we can use the `ls` command's `-l` option to view the permissions (or **file mode**) set for the contents of a directory:

```shell
ls -l /tmp/missing/semester
```

> -rw-rw-r-- 1 xyu xyu 61 Sep 10 21:09 /tmp/missing/semester

**owner/group/others**

### 7

```shell
sh /tmp/missing/semester
```

> Why does this work, while `./semester` didn’t?

When we type commands in shell, the first word is seen as the name of executable (if it is not absolute or relative path，shell will seek it in system path). However, we know that the _semester_ script is not executable from the permissions listed above while `sh` is executable we can found in system path. For `sh` to "execute" the script, it only needs to be able to read the file.

### 8

```shell
man chmod
```

### 9

```shell
chmod + x/tmp/missing/semester
```

> -rwxrwxr-x 1 xyu xyu 61 Sep 10 21:09 /tmp/missing/semester

Now the script is executable.

### 10

```shell
/tmp/missing/semester | sed -n '6p' > ~/last-modified.txt
cat ~/last-modified.txt
```

> last-modified: Sat, 03 Sep 2022 12:25:26 GMT

###  11

```shell
sudo find -L /sys/class/power_supply -maxdepth 3 -name 'BAT*'
cat /sys/class/power_supply/BAT0/capacity_level
cat /sys/class/thermal/thermal_zone*/temp
```