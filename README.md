# An introduction to fuzzing with American Fuzzy Lop

Here's the [presentation](https://docs.google.com/presentation/d/1pAuq16LorXcDpxPHfEj-f_mAyTCCzTKTBtQHKLaq060/edit?usp=sharing).

The prerequisite for the session would be to complete the steps from below till `"Installing QEMU mode"`.

## **Overview**

1. [Installing Prerequisites](#Installing-Prerequisites)
2. [Installing AFL](#-Installing-AFL)
3. [Working with AFL](#Working-with-AFL)
4. [Miscellaneous](#Miscellaneous)
5. [Hands-on](#Hands-on)
6. [Optimising the fuzzing process](#Optimising-the-fuzzing-process)
7. [Fuzzing binaries without source](#Fuzzing-binaries-without-source)

## **Installing Prerequisites**

* Install required compilers with the following commands:

```bash
sudo apt install gcc
sudo apt install clang
```

* Install `GDB` with the following command:

```bash
sudo apt install gdb
```

* Install `exploitable` with the following commands:

```bash
git clone https://github.com/jfoote/exploitable.git
cd exploitable/
python setup.py install
```

* Install `screen` with the following command:

```bash
sudo apt install screen
```

* To run `QEMU` mode, we'd need to install a bunch of dependencies. Install the dependencies by running the following commands:

```bash
sudo apt install libtool-bin
sudo apt install automake
sudo apt install bison
sudo apt install libglib2.0-dev
sudo apt install qemu
```

## **Installing AFL**

1. Install `AFL` with these commands:

```bash
wget http://lcamtuf.coredump.cx/afl/releases/afl-latest.tgz
tar -xzvf afl-latest.tgz
cd afl-2.52b/
make
sudo make install
```

2. Install llvm compiler with these commands:

```bash
cd afl-2.51b/llvm_mode/
sudo apt-get install llvm-dev llvm
make
cd ..
make
sudo make install
```

3. Install QEMU mode with the following commands:

```bash
cd afl-2.52b/qemu_mode
./build_qemu_support.sh
cd ..
sudo make install
```

## **Working with AFL**

1. Compile the application with the following commands:

```bash
export CC=afl-clang-fast
export AFL_HARDEN=1
export AFL_INST_RATIO=100
./configure
make
```

2. Build `test corpus` witht the following command:

```bash
cp /bin/ps afl_in/
```

3. Download `binutils` (or any binary):

```bash
wget http://ftp.gnu.org/gnu/binutils/binutils-2.25.tar.gz
```

4. Build binary for `binutils`:

```bash
tar -xvzf binutils-2.25.tar.gz
cd ~/binutils-2.25
CC=afl-clang-fast ./configure
make
```

5. System configuration change to avoid false-negatives:

```bash
sudo bash -c "echo core > /proc/sys/kernel/core_pattern"
```

6. Build required directories for AFL with the following commands:

```bash
cd ~/binutils-2.25
mkdir afl_in afl_out
cp /bin/ps afl_in/
```

7. Start fuzzing with the following command:

```bash
cd ~/binutils-2.25
afl-fuzz -i afl_in -o afl_out -- ./binutils/readelf -a @@
```

## **Miscellaneous**

* To check available cores use the following command:

```bash
afl-gotcpu
```

* To run parallel fuzzers on `binutils` with `screen`, use the following commands:

```bash
screen -dmS fuzzer1 /bin/bash -c "afl-fuzz -i afl_in -o alf_out -M fuzzer1 -- ./binutils/readelf -a @@"
screen -dmS fuzzer2 /bin/bash -c "afl-fuzz -i afl_in -o alf_out -S fuzzer2 -- ./binutils/readelf -a @@"
screen -dmS fuzzer3 /bin/bash -c "afl-fuzz -i afl_in -o alf_out -S fuzzer3 -- ./binutils/readelf -a @@"
screen -dmS fuzzer4 /bin/bash -c "afl-fuzz -i afl_in -o alf_out -S fuzzer4 -- ./binutils/readelf -a @@"
```

* To read from the specified fuzzer, use the following command:

```bash
screen -rd <session name>
```

* To detach from a `screen` session and return back to the terminal, use the following key combination:

```keyboard
Ctrl + a
d
```

## **Hands-on**

1. Clone `fuzzgoat` with the following command:

```bash
git clone https://github.com/fuzzstati0n/fuzzgoat
```

2. Compile `fuzzgoat` with the following command:

```bash
cd fuzzgoat
CC=afl-clang-fast
make
```

3. To make required directories for `fuzzgoat` (it already has a `input-files` directory), use the following command:

```bash
mkdir afl_out
```

4. To starting the fuzzers in parallel with `screen`, use the following commands:

```bash
screen -dmS fuzzer1 /bin/bash -c "afl-fuzz -i input-files -o alf_out -M fuzzer1 -- ./fuzzgoat @@"
screen -dmS fuzzer2 /bin/bash -c "afl-fuzz -i input-files -o alf_out -S fuzzer2 -- ./fuzzgoat @@"
screen -dmS fuzzer3 /bin/bash -c "afl-fuzz -i input-files -o alf_out -S fuzzer3 -- ./fuzzgoat @@"
screen -dmS fuzzer4 /bin/bash -c "afl-fuzz -i input-files -o alf_out -S fuzzer4 -- ./fuzzgoat @@"
```

5. To reading  AFL output, use the following command:

```bash
screen -rd fuzzer1
```

6. To check status of fuzzers, use the following command:

```bash
afl-whatsup afl_out
```

7. To examine crash with `GDB`, use the following command:

```bash
gdb ../../../fuzzgoat
```

8. To check for exploitable bug, use the following command:

```bash
(gdb) source ../../../../../exploitable/exploitable/exploitable.py

    --- snipped ---

(gdb) exploitable
```

## **Optimising the fuzzing process**

* To minimise the number of test cases, use the following command:

```bash
afl-cmin -i afl_in -o afl_out -- ./fuzzgoat @@
```

* To minimise the individual test cases, use the following command:

```bash
afl-tmin -i afl_in -o afl_out -- ./fuzzgoat @@
```

## **Fuzzing binaries without source**

* To fuzz binaries without source with QEMU mode, use the following command:

```bash
afl-fuzz -Q -i afl_in -o alf_out -- <Binary> <options> -a @@
```
