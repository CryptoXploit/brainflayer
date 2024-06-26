Brainflayer
===========

Brainflayer is a Proof-of-Concept brainwallet cracking tool that uses
[libsecp256k1](https://github.com/bitcoin/secp256k1) for pubkey generation.

Description
-----------


It was originally released as part of DEFCON talk about cracking brainwallets
([slides](https://rya.nc/dc23), [video](https://rya.nc/b6), [why](https://rya.nc/defcon-brainwallets.html)).

The name is a reference to [Mind Flayers](https://en.wikipedia.org/wiki/Illithid),
a race of monsters from the Dungeons & Dragons role-playing game. They eat
brains, psionically enslave people and look like lovecraftian horrors.

Disclaimer
----------

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS & CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.

Building
--------

Dependencies should install with

```
apt install autoconf openssl libgmp3-dev libimobiledevice-dev libplist-dev libusbmuxd-dev libssl-dev zlib1g-dev dh-autoreconf libltdl-dev libltdl7 libtool
```

Supported build target is currently Ubuntu 20.04 and above on amd64/x86_64. 

Usage
-----

### Basic

Precompute the bloom filter:

`hex2blf example.hex example.blf`

Run Brainflayer against it:

`brainflayer -v -b example.blf -i phraselist.txt`

or

`your_generator | brainflayer -v -b example.blf`

---
bitcoin puzzle
pip3 install bit
python3 puzzle.py < bit >
example for 66 bit puzzle :

python3 puzzle.py 66
0000000000000000000000000000000000000000000000034f7030c8aaf422e7
0000000000000000000000000000000000000000000000035d554e690b893e41
0000000000000000000000000000000000000000000000029eb02aa6cc332be6
0000000000000000000000000000000000000000000000028da2e6c49a202655
...
python3 puzzle 66 | ./brainflayer -v -c c -b 66.blf -t priv -x -o output.txt
rate:   9038.34 p/s found:     0/24576      elapsed:    2.594 s
...
mnemonics word
python3 mnemonics.py < number of mnemonics word >
example for 12 mnemonics word :

python3 mnemonics.py 12
picture waste cruel riot lady lift venture path erode sauce narrow expose
angry final exotic raccoon stable sibling start shoe leopard salon bag knife
wool horn jar only curtain sail journey fog come dentist boss aunt
brisk card hazard theme patch badge text slab absurd donate predict where
...
python3 mnemonics.py 12 | ./brainflayer -v -b btc-addess.blf -o output.txt
rate:  23744.46 p/s found:     0/49152      elapsed:    2.097 s
...
minikey
python3 minikey.py < 22 | 30 >
example for 22 minikey :

python3 minikey.py 22
SXmQMA5szCKoPkhfu3i8G4
S4G8i3ufhkPoKCzs5AMQmX
SsuTCqJDwVX536dFRyUkSn
SnSkUyRFd635XVwDJqCTus
...
python3 minikey.py 22 | ./brainflayer -v -b minikey.blf -o output.txt
rate:  23744.46 p/s found:     0/49152      elapsed:    2.097 s
...

### Advanced

Brainflayer's design is heavily influenced by [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy).
It (mostly) does one thing: hunt for tasty brainwallets. A major feature it
does *not* have is generating candidate passwords/passphrases. There are plenty
of other great tools that do that, and brainflayer is happy to have you pipe
their output to it.

Unfortunately, brainflayer is not currently multithreaded. If you want to have
it keep multiple cores busy, you'll have to come up with a way to distribute
the work yourself (brainflayer's -n and -k options may help). In my testing,
brainflayer benefits significantly from hyperthreading, so you may want to
run two copies per physical core. Also worth noting is that brainflayer mmaps
its data files in shared memory, so additional brainflayer processes do not
use up that much additional RAM.

While not strictly required, it is *highly* recommended to use the following
options:

* `-m FILE` Load the ecmult table from `FILE` (generated with `ecmtabgen`)
            rather than computing it on startup. This will allow multiple
            brainflayer processes to share the same table in memory, and
            signifigantly reduce startup time when using a large table.

* `-f FILE` Verify check bloom filter matches against `FILE`, a list of all
            hash160s generated with
            `sort -u example.hex | xxd -r -p > example.bin`
            Enough addresses exist on the Bitcoin network to cause false
            positives in the bloom filter, this option will suppress them.

Brainflayer supports a few other types of input via the `-t` option:

* `-t keccak` passphrases to be hashed with keccak256 (some ethereum tools)

* `-t priv` raw private keys - this can be used to support arbitrary
            deterministic wallet schemes via an external program. Any trailing
            data after the hex encoded private key will be included in
            brainflayer's output as well, for reference. See also the `-I`
            option if you want to crack a bunch of sequential keys, which has
            special speed optimizations.

* `-t warp` salts or passwords/passphrases for WarpWallet

* `-t bwio` salts or passwords/passphrases for brainwallet.io

* `-t bv2`  salts or passwords/passphrases for brainv2 - this one is *very* slow
            on CPU, however the parameter choices make it a great target for GPUs
            and FPGAs.

* `-t rush` passwords for password-protected rushwallets - pass the fragment (the
            part of the url after the #) using `-r`. Almost all wrong passwords
            will be rejected even without a bloom filter.

Address types can be specified with the `-c` option:

* `-c u` uncompressed addresses

* `-c c` compressed addresses

* `-c e` ethereum addresses

* `-c x` most signifigant bits of public point's x coordinate

It's possible to combine two or more of these, e.g. the default is `-c uc`.

An incremental private key brute force mode is available for fans of
[directory.io](http://www.directory.io/), try

`brainflayer -v -I 0000000000000000000000000000000000000000000000000000000000000001 -b example.blf`

See the output of `brainflayer -h` for more detailed usage info.

Also included is `blfchk` - you can pipe it hex encoded hash160 to check a
bloom filter file for. It's very fast - it can easily check millions of
hash160s per second. Not entirely sure what this is good for but I'm sure
you'll come up with something.


Authors
-------

The bulk of Brainflayer was written by Ryan Castellucci. Nicolas Courtois and
Guangyan Song contributed the code in `ec_pubkey_fast.c` which more than
doubles the speed of public key computations compared with the stock secp256k1
library from Bitcoin. This code uses a much larger table for ec multiplication
and optimized routines for ec addition and doubling.
