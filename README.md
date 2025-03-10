While this began just as a kind of adaptive index/hash table library, it has
grown into more a collection in the theme of database/big-data related data
structures & algorithms.  { Let's say the "ad" in "adix" now stands for
"ADvanced" or "AscenDant" now instead of "adaptive" ;-) }  Most of these are
just kind of *à la carte* and I hope you find them useful.  I try to keep the
source code short & to the point.  In particular, as an overview/index here be:

 - The original associative lookup modules:
   - [ditab](https://c-blake.github.io/adix/adix/ditab.html): direct-indexed
   table for small universe keys
   - [lptabz](https://c-blake.github.io/adix/adix/lptabz.html): a hash table
   that should be safe for untrusted key sets
   - [metab](https://c-blake.github.io/adix/adix/metab.html): an instantiation/
   table portability helper for lptabz
   - [btree](https://c-blake.github.io/adix/adix/btree.html): a B-Tree with
   various optional features (histogram, ranks, bulk loading, etc.)

 - Basic Sorting: [nsort](https://c-blake.github.io/adix/adix/nsort.html)
     Radix sort only by NEEDED BITS; Often 5-10X faster than `algorithm.sort` if
     you sort a lot of meso-scale data (merge sorts *always* win for HUGE data;
     Very few have it). (Could buffer writes to ensure full cache-line pokes.)

 - Basic Sketches (abbreviated/approximate stores; aka "Digests") for:
   - Membership: [bltab](https://c-blake.github.io/adix/adix/bltab.html) (bit-
   level table; Like more successfully marketed Bloom|Cuckoo filters, but lower
   latency & slightly bigger)
   - Count Distinct: [uniqce](https://c-blake.github.io/adix/adix/uniqce.html)
   aka count unique or cardinality estimation
   - Approx Most Often: [amoft](https://c-blake.github.io/adix/adix/amoft.html)
   (aka approximate top-K most frequent | heavy-hitters)
   - Quantiles: [lghisto](https://c-blake.github.io/adix/adix/lghisto.html) or
   [tdigest](https://c-blake.github.io/adix/adix/tdigest.html) (for slower more
   accurate tail quantiles)
   - An assimilation: [`stat`](https://c-blake.github.io/adix/adix/stat.html)
   that works like `std/stats` but supports `del`, i.e. sliding/moving windows
   over data streams (like a moving average) as well as moving or running
   quantiles via `lghisto`.  Also includes bulk array stats that in some compile
   modes get fully SIMD vectorized inner loops.

 - And a few utility modules:
  - [althash](https://c-blake.github.io/adix/adix/althash.html): an assortment
  of salt-able alternate hash functions for lptabz
  - [xlang](https://c-blake.github.io/adix/adix/xlang.html): little definitions
  for C refugees for bist
  - [sequint](https://c-blake.github.io/adix/adix/sequint.html): a fixed stride
  "bit matrix" for bltab that uses "batch" number operations (unlike how a naive
  bit matrix over a bit vector would work).
  - [memutil](https://c-blake.github.io/adix/adix/memutil.html): memory shifting
  utilities for lptabz
  - [cumsum](https://c-blake.github.io/adix/adix/cumsum.html): parallel prefix
  sum using Intel SIMD for nsort
  - [bitop](https://c-blake.github.io/adix/adix/bitop.html): re-impl some
  std/bitops things to be more CT friendly

A Word Of Caution
=================
While sketches are popular, like merge sort (vs. radix sort), they often need
huge data to pay off.  Essentially, probabilistic analysis ("Oh wow, I can do
that?!") tends to distract from space-time trade offs.  This distraction is
worsened by there being space-time-accuracy "trade off pyramids".  So, I can say
here "spending a bit more space can yield major speed-ups", and it sounds
blatantly obvious to even the most casual observer.  Yet, I have also seen it
neglected in this context countless times.  The academic literature does not
help, often being "blood sport" for more compressed data | accuracy.

So, e.g., on my primary 32 GiB RAM dev box with `bu/zipf`, I cannot make exact
`lfreq` slower than Approximately Most Often sketches(`bu/oft`).  `tests/bl.nim`
shows another example in the Bloom filter / membership approximation context.

A little more on LPTabz & friends
=================================
As a brief guide I would start with `NOTES.md` and then look at the top half of
`lptabz.nim`.  `TODO.md` also has a lot of notes in it.  My overarching vision
is to allow "the fast way" most of the time, especially for developers that know
how to provide a good `hash`, but to also have auto fall backs to "safer ways"
with optional messages to let devs know they may need to intervene by changing
some defaults at table construction time (or else let users/sysadmins know that
some input may be violating the assumptions of some code sensitive to inputs).
Commercial database systems may have done this for decades, but hasn't really
percolated into commonly available runtime libs.  (Depth-based growth trigger is
likely the simplest example of Profile-Guided Optimization for data structures.
A.Dain Samples 1993 PhD thesis has some more.)
