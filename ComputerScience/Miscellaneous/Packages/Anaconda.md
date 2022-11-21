## Install

To Install `Anaconda` in Ubuntu 22.04, please refer to [linuxhint](https://linuxhint.com/install-anaconda-ubuntu-22-04/).

The installer has initialized the `Anacoda3` in _~/.bashrc_:

```shell
# added by Anaconda3 5.3.1 installer
# >>> conda init >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$(CONDA_REPORT_ERRORS=false '/home/xyu/anaconda3/bin/conda' shell.bash hook 2> /dev/null)"
if [ $? -eq 0 ]; then
    \eval "$__conda_setup"
else
    if [ -f "/home/xyu/anaconda3/etc/profile.d/conda.sh" ]; then
        . "/home/xyu/anaconda3/etc/profile.d/conda.sh"
        CONDA_CHANGEPS1=false conda activate base
    else
        \export PATH="/home/xyu/anaconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda init <<<
```

I pasted the code to my *~/.bash_profile* in order to make my `tmux` also initialize the `Anacoda3`.

## Basic Usage

Check and update conda:

```shell
conda update conda
```

Viewing a list of installed packages:

```shell
conda list
```

Viewing a list of existed virtual environment:

```shell
conda env list
conda info -e
```

Create Python virtual environment:

```shell
conda create -n $your_env_name python=x.x
```

I checked [Active Python Release](https://www.python.org/downloads/), then created environment of `3.10`, `3.9`, `3.8` and `3.7` versions.

To activate a environment:

```shell
conda activate $your_env_name
```

To deactivate an active environment:

```shell
conda deactivate
```

To remove a environment:

```shell
conda remove -name $your_env_name --all
```

To remove some package of a environment:

```shell
conda remove -name $your_env_name $package_name
```

To install a package into your current environment:

```shell
conda install $package_name
```

To install specific a specific version of a package:

```shell
conda install $package_name-$version
```

We can install multiple packages at once.

**It is recommended to install all required packages at once so that all of the dependencies are installed at once.**

To install a specific package into your existing environment `your_env_name`:

```shell
conda install –name $your_env_name $package_name
```

If the package is not available in our conda environment , we can find and install the package with another package manager like `pip`.

We can install `pip` in our existing conda environment by simply giving the command:

```shell
conda install pip
```

## Problems

[VS Code not detecting package in conda environment](https://stackoverflow.com/questions/63484377/vs-code-not-detecting-package-in-conda-environment)