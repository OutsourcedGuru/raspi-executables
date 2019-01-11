# A step-by-step guide for creating executable programs which will run on a Raspberry Pi 3B
Sometimes you need to hide the details of distributable programs which provide functionality on Raspbian. For example, you might have a program which encrypts/decrypts data and you don't necessarily want to make the source available as a result.

### Requirements
All the examples below assume an ARMv7 architecture as found on a Raspberry Pi 3B, 3B+ or A+ models. The Raspberry Pi Zero W, for example, is an ARMv6 architecture so adjustments to the instructions would be necessary (though not included below).

> If you compile on a Raspberry Pi Zero W, the executable can be distributed to and run on a Raspberry Pi 3B (at least, in theory). It is rumored that the reverse is true, noting that *your mileage may vary*.

### Compile-here-use-elsewhere strategy
Throughout all these examples it is to be noted that you'll compile on one Raspberry Pi and distribute just the resulting executable for other Raspberry Pi computers. (You won't need to install anything else on the receiving computer.)

## Compiled Python
One of the easiest methods of hiding the details of your code would be to pre-compile a Python script and to only deliver that version.

> It would be nice to use **pyinstaller** but it doesn't seem to include a pre-compiled bootloader for ARMv7, for what it's worth. I'm taking the Cython route here. I'm assuming Python2.7 throughout.

### On your Raspberry Pi 3B:

1. Begin with a standard Raspbian installation
2. Remote into the Pi using `ssh`, for example
3. `mkdir -p ~/scripts/hello-py-src && cd ~/scripts/hello-py-src`
4. `touch hello.py && nano hello.py`

> Note the use of single quotes to surround a string.

```
#!/usr/bin/env python

print 'Hello, World!\n'
```

> If this is running as a script within an OctoPrint-imaged Pi, the first line needs to instead be `#!/home/pi/oprint/bin/python`

&nbsp;5. [Required only once]: `pip --version`

&nbsp;6. [Required only once]: If **pip** is not installed yet: `sudo apt-get install -y python-pip`

> If this is running as a script within OctoPrint and you didn't find **pip**, you should stop and run `source ~/oprint/bin/activate` to enter the virtual environment and then repeat the last `pip --version` command, continuing from here without attempting to install **pip**.

&nbsp;7. [Required only once]: `pip install Cython`

&nbsp;8. [Required only once]: `sudo apt-get install -y python2.7-dev`

&nbsp;9. `~/.local/bin/cython -2 --embed -o hello.c hello.py`

> Note that I'm indicating the location of **cython** since it didn't happen to be in my $PATH variable.

&nbsp;10. `cc -Os -I /usr/include/python2.7 -o hello hello.c -lpython2.7 -lpthread -lm -lutil -ldl`

&nbsp;11. `cp hello ..`

&nbsp;12. `cd .. && ls -l`

```
-rwxr-xr-x 1 pi pi 15216 Jan 10 20:16 hello          # = 15KB
drwxr-xr-x 2 pi pi  4096 Jan 10 20:14 hello-py-src
```

&nbsp;13. `./hello`

```
Hello, World!
```

So now you have an executable program which was originally a Python script, compiled to a C program and finally linked into an executable. **You may now distribute just the stand-alone `hello` program which will run on any ARMv7 architecture computer like another Raspberry Pi 3.**

You do not need the original `hello-py-src` folder or any of its contents for distribution. You do not need to install anything else on the receiving Raspberry Pi computer.

## Compiled C/C++
It's easy enough to compile a C or C++ script on Raspian and then just distribute the executable version of same.

### On your Raspberry Pi 3B:

1. Begin with a standard Raspbian installation
2. Remote into the Pi using `ssh`, for example
3. `mkdir -p ~/scripts/hello-c-src && cd ~/scripts/hello-c-src`
4. `touch hello.c && nano hello.c`

> Note the use of double quotes in C to surround a string.

```
#include <stdio.h>

void main() {
    printf("Hello, World!\n");
}
```

&nbsp;5. `gcc -o hello-c hello.c`

&nbsp;6. `cp hello-c .. && cd ..`

&nbsp;7. `ls -l`

```
-rwxr-xr-x 1 pi pi  8148 Jan 10 20:41 hello-c          # = 8KB
drwxr-xr-x 2 pi pi  4096 Jan 10 20:40 hello-c-src
```

&nbsp;8. `./hello-c`

```
Hello, World!
```

So now you have an executable program as in the previous example. **You may now distribute just the stand-alone `hello-c` program which will run on any ARMv7 architecture computer like another Raspberry Pi 3.**

You do not need the original `hello-c-src` folder or any of its contents for distribution. You do not need to install anything else on the receiving Raspberry Pi computer.

## Compiled Go language
The Go language is relatively new and compiles into an executable.

### On your Raspberry Pi 3B:

1. Begin with a standard Raspbian installation
2. Remote into the Pi using `ssh`, for example
3. [Required only once]: `mkdir ~/tmp && cd ~/tmp`
4. [Required only once]: `wget https://dl.google.com/go/go1.10.7.linux-armv6l.tar.gz`
5. [Required only once]: `sudo tar -C /usr/local -xzf go1.10.7.linux-armv6l.tar.gz`
6. [Required only once]: `nano ~/.profile`

#### Add to end of file:

```
export PATH=$PATH:/usr/local/go/bin
```

&nbsp;7. [Required only once]: `source ~/.profile`

&nbsp;8. [Required only once]: `go version`

&nbsp;9. `mkdir ~/scripts/hello-go-src && cd ~/scripts/hello-go-src`

&nbsp;10. [Required only once]: `rm -Rf ~/tmp`

&nbsp;11. `touch hello.go && nano hello.go`

```
package main
import (
	"fmt"
)

func main() {
	fmt.Printf("Hello, World!\n")
	return
}
```

&nbsp;12. `go build`

&nbsp;13. `cp hello-go-src ../hello-go && cd ..`

&nbsp;14. `ls -l`

```
drwxr-xr-x 2 pi pi    4096 Jan 10 20:59 hello-go-src
-rwxr-xr-x 1 pi pi 1826069 Jan 10 21:02 hello-go          # = 1.8MB
```

So now you have an executable program as in the previous example. **You may now distribute just the stand-alone `hello-go` program which will run on any ARMv7 architecture computer like another Raspberry Pi 3.**

You do not need the original `hello-go-src` folder or any of its contents for distribution. You do not need to install anything else on the receiving Raspberry Pi computer.

## Compiled NodeJS (Javascript) with pkg
Zeit.co provides a great Node compiler, allowing you to distribute just an executable out of your JavaScript code.

### On your Raspberry Pi 3B:

> Dependency: You must first install `NodeJS` and `npm` for ARMv7 on your Raspberry Pi.

1. Begin with a standard Raspbian installation
2. Remote into the Pi using `ssh`, for example
3. [Required only once]: `mkdir ~/tmp && cd ~/tmp`
4. [Required only once]: `wget https://nodejs.org/dist/v8.9.3/node-v8.9.3-linux-armv7l.tar.xz`
5. [Required only once]: `tar -xvf node-v8.9.3-linux-armv7l.tar.xz`
6. [Required only once]: `cd node-v8.9.3-linux-armv7l`
7. [Required only once]: `sudo cp -R * /usr/local/`
8. [Required only once]: `node --version && npm --version`
9. [Required only once]: `sudo npm install -g pkg`
10. `mkdir ~/scripts/hello-js-src && cd ~/scripts/hello-js-src`
11. [Required only once]: `rm -Rf ~/tmp`
12. `touch hello.js && nano hello.js`

```
#!/usr/bin/env node

console.log('Hello, World!\n');
```

&nbsp;13. `pkg -t node8-linux-armv7 -o hello-js hello.js`

&nbsp;14. `cp hello-js .. && cd ..`

&nbsp;15. `ls -l`

```
-rwxr-xr-x 1 pi pi 31303361 Jan 10 21:20 hello-js          # = 31.3MB
drwxr-xr-x 2 pi pi     4096 Jan 10 21:19 hello-js-src
```

So now you have an executable program as in the previous example. **You may now distribute just the stand-alone `hello-js` program which will run on any ARMv7 architecture computer like another Raspberry Pi 3.**

You do not need the original `hello-js-src` folder or any of its contents for distribution. You do not need to install anything else on the receiving Raspberry Pi computer.

# Comparison
Here's a table of the executable file sizes as produced from the four `Hello, World!` versions (plus an additional C++ alternative).

|Version|Size in bytes|Comments|
|:---|---:|:---|
| C | 8,148 | Obviously, this is the smallest executable of the collection here. Note that the difference versus a C++ file would have been similarly-sized. |
| C++ | 8,416 | Almost as small as C. Note the differences in the C example: 1) The source file should end in `.cpp`, 2) the main function needs to be changed to `int main()`, 3) the return command becomes `return 0;`, 4) the compile command becomes `gcc -o hello-cpp hello.cpp`. |
| Python | 15,216 | This isn't too bad as file size goes. Given the compilation step to a C object file as an intermediate step, this is to be expected. |
| Go | 1,826,069 | This comes in fairly heavy by comparison, noting that the Go language has a rich collection of packages. |
| NodeJS | 31,303,361 | Although this weighs in as the heaviest file, the NodeJS engine (V8) allows for some pretty sophisticated programming and the ease, familiarity and convenience of JavaScript programming.|


---

|Description|Version|Author|Last Update|
|:---|:---|:---|:---|
|raspi-executables|v1.0.2|OutsourcedGuru|January 11, 2019|

|Donate||Cryptocurrency|
|:-----:|---|:--------:|
| ![eth-receive](https://user-images.githubusercontent.com/15971213/40564950-932d4d10-601f-11e8-90f0-459f8b32f01c.png) || ![btc-receive](https://user-images.githubusercontent.com/15971213/40564971-a2826002-601f-11e8-8d5e-eeb35ab53300.png) |
|Ethereum||Bitcoin|
