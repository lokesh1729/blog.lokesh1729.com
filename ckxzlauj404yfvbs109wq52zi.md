## How to install python on Mac M1

Hey there!

How are you doing ? Welcome to another blog post. In this post, I'll guide you on how to install python on Mac M1 (apple silicon chip). Before we get started, let me tell you why is it difficult to install python on the M1 chip. 

### The difference in computer architectures

There are many different ways the computer can be architectured. CPUs work by executing assembly language. Everything will be in binary and there will be instructions such as ADD, LSHIFT, RSHIFT etc... So, each architecture is different in processing these instructions.

**x86 & x64** - This is the most commonly used architecture. Most of the intel CPUs use this architecture.

**arm64** - Apple silicon chip uses this architecture. This is very popular among mobiles, raspberry pi etc..

### How to install python on Mac M1

Most of the software works very well on x86 & x64 architectures because it is very old and popular. The same case with `python` too. We don't have build tools and processes for `arm64` architecture. Hence, we need a translator which converts the `x86`/`x64` instructions from `python` build tools to `arm64` instructions. It is called `rosetta`. It is developed by Apple to give backward compatibility for the users.

So, we need to install it first.

**Step 1 - Install Rosetta**

Open a terminal and copy-paste the below command

```bash
/usr/sbin/softwareupdate --install-rosetta --agree-to-license
``` 

**Step 2 - Install XCode developer tools**

```bash
xcode-select --install
```

**Step 3 - Install homebrew**

It is a package manager for installing new software.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Step 4 - Rosetta Terminal**

Go to your “Applications” folder on Finder → right-click Terminal in the “Utilities” folder → Duplicate → rename to “Rosetta Terminal” → Get Info → Open using Rosetta

> Pro tip: install iTerm2 using `brew install iterm2`. It is terminal on steroids. It has so many features such as horizontal and vertical splitting, autocompletion, etc...

If you are using iTerm2 then follow a similar process for it too.

Go to your “Applications” folder on Finder → right-click iTerm2 → Duplicate → rename to “Rosetta iTerm2” → Get Info → Open using Rosetta

**Step 5 - Install homebrew on Rosetta Terminal**

Open "Rosetta iTerm2" and install homebrew again. This is x86 / x64 version of homebrew installed on top of rosetta. Repeat step 3 again.

**Step 6 - Install Pyenv**

To install python we will use `pyenv`. Let's install it first

Copy and paste this command on **Rosetta iTerm2**

```bash
curl https://pyenv.run | bash
```

**Step 7 - Install Python using pyenv**

**Execute these commands on Rosetta iTerm2**

Install some pre-requisites first

```bash
brew install openssl@3 bzip2 readline zlib 
```

Let's install python now.

For python >= 3.8.*

**For versions 3.8.3 and 3.8.7 we can use patches from homebrew**

You can find the patches [here](https://github.com/Homebrew/formula-patches/tree/master/python) 

```bash
pyenv install --patch 3.8.7 <<(curl -sSL https://raw.githubusercontent.com/Homebrew/formula-patches/master/python/3.8.7.patch\?full_index\=1)
```

For any other versions or if the above patch doesn't work, try with patch from `cpython`. Just replace the version 3.8.2 with other version.

```bash
CFLAGS="-I$(brew --prefix openssl)/include -I$(brew --prefix bzip2)/include -I$(brew --prefix readline)/include -I$(xcrun --show-sdk-path)/usr/include" LDFLAGS="-L$(brew --prefix openssl)/lib -L$(brew --prefix readline)/lib -L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" pyenv install --verbose --patch 3.8.2 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)
```

That's all folks! in my next post, I'll write an article on setting up virtual environments in python. See you till then, Bye!
