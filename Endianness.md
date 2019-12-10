# JavaScript and endianess

JavaScript is a high-level, interpreted language and for a while the programmer
was not bothered with concepts like endianness. Now this is not true anymore.
JavaScript has byte buffers, so it is important to grasp some concept to be
able to work with them.

Let us try to investigate the properties of NaNs in JavaScript a little bit.
NaN is a value of a numeric type and represents the result of a computation
that cannot be performed, hence "Not A Number". In double-floating point
arithmetics there are multiple ways to represent a NaN, but let us go through
some theory first.

A double float consists of 64 bits: the first bit is the sign of the number,
followed by 11 bits reserved for the exponent, and then the rest 52 bits are
assigned to the significand. Every value that has all exponent bits set to one
and a non-zero significand represents a NaN. The sign bit is irrelevant. That
means that there are 2^53-2 values for NaN. The first bit of the significand
differentiates between the so called "quiet" and "signalling" NaN.

In JavaScript there is just one single NaN value regardless of the underlying
representation. However thanks to the typed arrays we can try to uncover the
real value. We will create a byte buffer in JavaScript and then apply
different typed arrays to map between number types:

```js
const a = new Float64Array(1)
const b = new Uint8Array(a.buffer)

console.log(a.length) // 1
console.log(b.length) // 8
```

Those two arrays map onto the same byte buffer but represent 1 double-float
value which is of 8-byte size, or 8 unsigned byte values which are of 1-byte
size. While writing to the float array we can get the exact byte values by
reading from the unsigned byte array. So let's try:

```js
a[0] = NaN
console.log(b) // [ 0, 0, 0, 0, 0, 0, 248, 127 ]
```

What happened here ? There are 6 bytes with zero values and 2 bytes with
non-zero values. 127 corresponds to eight most-significant bits, which are the
sign bit and seven bits of the exponent. The sign bit is equal to zero, and the
seven exponent bits are set to one. 248 represents four least-significant bits
of the exponent and four most-significant of the significand. First five bits
are set to one, because 248 = 128 + 64 + 32 + 16 + 8. And here are the first
two bytes in double representation:

```js
0 11111111111 100...
```

That means that the NaN in JavaScript is the quiet NaN, because the first bit
of the significand is set to one. Now let's try some more experimentation:

```js
a[0] = 0 / 0
console.log(b) // [ 0, 0, 0, 0, 0, 0, 248, 127 ]

a[0] = Infinity / Infinity 
console.log(b) // [ 0, 0, 0, 0, 0, 0, 248, 255 ]
```

Here we can clearly produce two distinct NaN values by applying two different
arithmetic operations that lead to an undefined value. However in JavaScript
those two different values are equal.

```js
Object.is(0 / 0, Infinity / Infinity) // true
```

This conforms to the ECMAScript specification that says:

> In some implementations, external code might be able to detect a difference
> between various Not-a-Number values, but such behaviour is
> implementation-dependent; to ECMAScript code, all NaN values are
> indistinguishable from each other.

Now what with the concept of endianness ? Note that the byte values are in the
reverse order, i.e. the least-significant byte in the word comes first, but we
would expect it to come last. Why is it so ? Different computer architectures
define a different order of bytes in a word.  It is a processor-related thing.
In the so called "little endian" the least-significant byte comes first. Intel
and ARM are little endian. The TCP protocol defines that the byte order be in
big endian. PowerPC is also big endian, so if you are using a PowerPC-based
computer, you might not have been able to repeat the above experiments exactly.

