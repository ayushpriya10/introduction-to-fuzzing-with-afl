# An introduction to fuzzing with American Fuzzy Lop

Here's the [presentation](https://docs.google.com/presentation/d/1pAuq16LorXcDpxPHfEj-f_mAyTCCzTKTBtQHKLaq060/edit?usp=sharing)

The prerequisite for the session would be to complete the steps from below till `"Installing QEMU mode"`.

* Installing compilers:

```bash
sudo apt install gcc
sudo apt install clang
```

* Installing `GDB`:

```bash
sudo apt install gdb
```

* Installing `exploitable`:

```bash
git clone https://github.com/jfoote/exploitable.git
cd exploitable/
python setup.py install
```

* Installing `screen`:

```bash
sudo apt install screen
```

* To run `QEMU` mode, we'd need to install a bunch of dependencies:

```bash
sudo apt install libtool-bin
sudo apt install automake
sudo apt install bison
sudo apt install libglib2.0-dev
sudo apt install qemu
```

* Installing `AFL`:

```bash
wget http://lcamtuf.coredump.cx/afl/releases/afl-latest.tgz
tar -xzvf afl-latest.tgz
cd afl-2.52b/
make
sudo make install
```

* Installing llvm compiler:

```bash
cd afl-2.51b/llvm_mode/
sudo apt-get install llvm-dev llvm
make
cd ..
make
sudo make install
```

* Installing QEMU mode:

```bash
cd afl-2.52b/qemu_mode
./build_qemu_support.sh
cd ..
sudo make install
```

* Compiling the application:

```bash
export CC=afl-clang-fast
export AFL_HARDEN=1
export AFL_INST_RATIO=100
./configure
make
```

* Building `test corpus`:

```bash
cp /bin/ps afl_in/
```

* Downloading `binutils`:

```bash
wget http://ftp.gnu.org/gnu/binutils/binutils-2.25.tar.gz
```

* Building binary for `binutils`:

```bash
tar -xvzf binutils-2.25.tar.gz
cd ~/binutils-2.25
CC=afl-clang-fast ./configure
make
```

* System configuration change to avoid false-negatives:

```bash
sudo bash -c "echo core > /proc/sys/kernel/core_pattern"
```

* Building required directories for AFL:

```bash
cd ~/binutils-2.25
mkdir afl_in afl_out
cp /bin/ps afl_in/
```

* Fuzzing:

```bash
cd ~/binutils-2.25
afl-fuzz -i afl_in -o afl_out -- ./binutils/readelf -a @@
```

* Check available cores:

```bash
afl-gotcpu
```

* Running parallel fuzzers on `binutils` with `screen`:

```bash
screen -dmS fuzzer1 /bin/bash -c "afl-fuzz -i afl_in -o alf_out -M fuzzer1 -- ./binutils/readelf -a @@"
screen -dmS fuzzer2 /bin/bash -c "afl-fuzz -i afl_in -o alf_out -S fuzzer2 -- ./binutils/readelf -a @@"
screen -dmS fuzzer3 /bin/bash -c "afl-fuzz -i afl_in -o alf_out -S fuzzer3 -- ./binutils/readelf -a @@"
screen -dmS fuzzer4 /bin/bash -c "afl-fuzz -i afl_in -o alf_out -S fuzzer4 -- ./binutils/readelf -a @@"
```

* To read from the specified fuzzer:

```bash
screen -rd <session name>
```

* To detach from a `screen` session and return back to the terminal:

```keyboard
Ctrl + a
d
```

* Clone `fuzzgoat`:

```bash
git clone https://github.com/fuzzstati0n/fuzzgoat
```

* Compiling `fuzzgoat`:

```bash
cd fuzzgoat
CC=afl-clang-fast
make
```

* Making required directories for `fuzzgoat` (it already has a `input-files` directory):

```bash
mkdir afl_out
```

* Starting the fuzzers in parallel with `screen`:

```bash
screen -dmS fuzzer1 /bin/bash -c "afl-fuzz -i input-files -o alf_out -M fuzzer1 -- ./fuzzgoat @@"
screen -dmS fuzzer2 /bin/bash -c "afl-fuzz -i input-files -o alf_out -S fuzzer2 -- ./fuzzgoat @@"
screen -dmS fuzzer3 /bin/bash -c "afl-fuzz -i input-files -o alf_out -S fuzzer3 -- ./fuzzgoat @@"
screen -dmS fuzzer4 /bin/bash -c "afl-fuzz -i input-files -o alf_out -S fuzzer4 -- ./fuzzgoat @@"
```

* Reading  AFL output:

```bash
screen -rd fuzzer1
```

* Check status of fuzzers:

```bash
afl-whatsup afl_out
```

* Examining crash with `GDB`:

```bash
gdb ../../../fuzzgoat
```

* Checking for exploitable bug:

```bash
(gdb) source ../../../../../exploitable/exploitable/exploitable.py

    --- snipped ---

(gdb) exploitable
```

* Minimising the number of test cases:

```bash
afl-cmin -i afl_in -o afl_out -- ./fuzzgoat @@
```

* Minimising individual test cases:

```bash
afl-tmin -i afl_in -o afl_out -- ./fuzzgoat @@
```

* Fuzzing binaries without source with QEMU mode:

```bash
afl-fuzz -Q -i afl_in -o alf_out -- <Binary> <options> -a @@
```
