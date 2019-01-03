# libraryofbabel.info-algo
Invertible multi-precision pseudo random number generator example

by Jonathan Basile, uploaded 1/2/2019
CC BY-SA-NC
No liability

I agonized for a long time about trying to share this code in the best possible form and it got to the point where obviously that was stopping me from sharing it at all, so here it is without much explanation (my apologies). I've shared the algorithm for babelia.libraryofbabel.info because it is significantly more efficient than the one I wrote for libraryofbabel.info. 

You'll need a lot of libraries installed if you want this code to work as it's written - the most important are gnu cgicc, boost multiprecision, gnu gmp, and ImageMagick

There's a lot to read through but basically it boils down to these lines:

*pointer = (a*(*pointer)+c)%m;   

*pointer ^= (*pointer >> 1098239);

*pointer ^=((*pointer%maskone) << 698879);

*pointer ^=((*pointer%masktwo) << 1497599);

*pointer ^=(*pointer >> 1797118);



and the inversion:

*pointer ^=(*pointer >> 1797118);

*revp = *pointer^((*pointer % masktwo) << 1497599);

*pointer = *pointer^((*revp % masktwo) << 1497599);

*revp = *pointer^((*pointer % maskone) << 698879);

*revp = *pointer^((*revp %maskone) << 698879);

*revp = *pointer^((*revp %maskone) << 698879);

*revp = *pointer^((*revp %maskone) << 698879);

*pointer = *pointer^((*revp %maskone) << 698879);

*revp = *pointer^(*pointer >> 1098239);

*revp = *pointer^(*revp >> 1098239);

*pointer = *pointer^(*revp >> 1098239);

*pointer = (ainverse*(*pointer-c))%m;

if (*pointer<0) {*pointer += m;}

This is a combination of a linear congruential generator and a mersenne twister (sort of) if you want to read more about them.

Of course, the code here can only be copy-pasted if you are dealing with a permutation set of exactly the same size. If you would like to use this algorithm to permute other things (please do! Permute everything! And share with me what you make! jonathan.e.basile [at] gmail.com) here's what you need to do:

For the LCG:

next = (a(x) + c) % m

x = ainverse(next-c) % m

m should be a number greater than the total number of possible permutations in your set (wolfram alpha is useful for figuring this out: https://www.wolframalpha.com). It should not be a power of the same base as your permutation set (i.e. if you are permuting a set of 32 elements, do not make m a power of 2).

Also, follow the Hull–Dobell Theorem:

1. m and c must be relatively prime
2. a-1 must be divisible by all prime factors of m
3. a-1 is divisible by 4 if m is divisible by 4

Pick values fairly close to but less than m for a and c, or else adjacent permutations won't appear particularly random.

ainverse needs to be found using Euclid's extended algorithm. I have uploaded code for this.

The bit-shifting operations are a little trickier to invert. I start with this:

unsigned int temper(unsigned int x)

   {
   x ^= (x >> 11);
   
   x ^= (x << 7) & 0x9D2C5680;
   
   x ^= (x << 15) & 0xEFC60000;
   
   x ^= (x >> 18);
   
   return x;
   }
The inversion function is:

unsigned int detemper(unsigned int x)
   {
   x ^= (x >> 18);
   
   x ^= (x << 15) & 0xEFC60000;
   
   x ^= (x << 7) & 0x1680;
   
   x ^= (x << 7) & 0xC4000;
   
   x ^= (x << 7) & 0xD200000;
   
   x ^= (x << 7) & 0x90000000;
   
   x ^= (x >> 11) & 0xFFC00000;
   
   x ^= (x >> 11) & 0x3FF800;
   
   x ^= (x >> 11) & 0x7FF;

   return x;
   }

I think these are the values for a 32-bit implementation, so take your value for m, convert it to a power of 2 (round the exponent up) and pick proportional values for each of the integers above, so:

m = 2^e

(11/32)*e would be the number of bits to shift in your first operation, etc.

maskone and masktwo in the algorithm I used are necessary because when shifting left you will be making the numbers larger, so you need to cut off an equivalent number of digits. Honestly, I don't remember how I figured out the correct values for maskone and masktwo.

I added the mersenne twister-like bit-shifting operations on top of the LCG because the LCG on its own did not create sufficiently random-seeming results. 
