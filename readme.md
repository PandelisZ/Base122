Compact Data URI
================

The goal is to improve the base64 data URI by using the ASCII characters that base64 neglects.

Existing Work
-------------
[Base91](http://base91.sourceforge.net/) implementation encodes 13-14 bit sequences in two 91 bit
characters. 91^2 = 8281 and 2^13 = 8192, so there is some room 8281-8192=89. The algorithm is:
Take 13 bits, if the value is < 88, than it is safe to consume an additional bit, since 2^13 + 88 <
8281. The final encoding uses ASCII characters
0xxxxxxx 0xxxxxxx with final values mapped from original data to a character table.

Two 91 bit characters can encode 13 bits, saving 1 bit for every 16 bits of input as opposed to
base64. The decoder is also probably a bit simpler.

Approach
--------

Based on quick testing (modern Edge, Chrome, Firefox, and IE) it seems like it is safe to pass
123 of the ASCII characters, excluding newline, double quote, carriage return, backslash, and null
character. However, null character seems fine to pass, but sublime text renders the HTML file as
a binary file.

This proposes to encode at least 7 bits per byte, improving base64 by 1 bit per byte and improving
base91 by about .5 bits per byte.

To handle values that appear outside of the base, i.e. numbers 124-127, I'm proposing to use two
byte UTF-8 characters to encode the 7 bits and consume additional information.

UTF-8 Encoding scheme
---------------------

If the 7 bits (x) are less than 123, map to our character table (y), and
encode the result.

1 byte: 0yyyyyyy (7 bits of information encoded in 8)

If the 7 bits are greater than or equal to 123 (123, 124, 125, 126, 127), encode the difference
128 - number in the first 3 bits (z) of a UTF-8 two byte character, and use the remaining bits
for more informatin (w)

2 bytes: 011zzzww 01wwwwww (15 bits of information encoded in 16)

Since the optimization with the 2 byte UTF-8 characters gives us a better ratio, it begs the
question of whether a smaller base will give us better overall compression since this will
only occur with small probability. A rough and likely incorrect formula for compression ratio is:

(x / 128) * 7 / 8 + ((128 - x) / 128) * (7 + 11 - ceil(log_2(128 - x))) / 16

Explanation:

x / 128 = probability of encoding in one byte
7 / 8 = compression ratio for one byte

(128 - x)/128 = probability of encoding in two bytes
7 + 11 - ceil(log_2(128 - x)) / 16 = compression ratio for two bytes

Briefly graphing this on my calculator gave me 122.11 as a maximum, so it seems decent.