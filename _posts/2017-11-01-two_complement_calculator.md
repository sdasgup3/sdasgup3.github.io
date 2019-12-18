---
layout: post
title: Decimal-Hexadecimal-2s Complement Binary Converter
tags:
- bits
- Number System
date: 01-11-2017
---

Getting rid of my dependecies on online convertors.
The online version coming soon...

```python
#!/usr/bin/python

##############################################################################################################
#   Extract information about a number.                                                                         #
#                                                                                                            #
#   Example Usage: python infonum.py --bit 4 0xf                                                             #
#   Output:                                                                                                  #
#           Base 10: -1                                                                                      #
#	    Base 16: f                                                                                       #
#	    2's Compliment binary: 1111                                                                      #
#   Example Usage: python infonum.py --bit 64 -1                                                             #
#   Output:                                                                                                  #
#	Base 10: -1                                                                                          #
#	Base 16: ff ff ff ff ff ff ff ff                                                                     #
#	2's Compliment binary: 11111111 11111111 11111111 11111111 11111111 11111111 11111111 11111111       #
#                                                                                                            #
##############################################################################################################


import argparse
import pdb
import re

BIT= int(64)
NEGATE = {'1': '0', '0': '1'}

# Convert a hex number (in 2's negate) to Decmal
def toDec(hexstr):
    msb4bits = hexstr[0]
    n = int(msb4bits, 16)
    if n >= 8:
        p = -1*pow(2,BIT-1)
        addend = int(str(n-8) + hexstr[1:], 16)
        return str( p + addend)
    else:
        return str(int(hexstr, 16))


# Convert a decimal number to  2's negate Hex
def toHex(n):

    num = int(n)
    if num == 0:
        return '0'

    M = '0123456789abcdef'  # like a map
    ans = ''

    chunks = int(BIT) / int(4)

    for i in range(chunks):
        n = num & 15       # this means num & 1111b
        c = M[n]          # get the hex char
        ans = c + ans
        num = num >> 4
    return ans

def twocomplement(n, size_in_bits):
    number = int(n)
    if number < 0:
        return negate(bin(abs(number) - 1)[2:]).rjust(size_in_bits, '1')
    else:
        return bin(number)[2:].rjust(size_in_bits, '0')

def negate(value):
    return ''.join(NEGATE[x] for x in value)

def prettybinary(value, separator):
    ans = ''
    for i in range(0,len(value)):
        if i != 0 and i % separator == 0:
            ans = ans + ' ' + value[i]
        else:
            ans = ans + value[i]
    return ans

if __name__ == "__main__":

    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--bit",
        default=64,
        help="Bit width of the number",
        required=True)
    parser.add_argument(
        "num", nargs='?',
        help="Number to be analyzed")

    args = parser.parse_args()
    if args.bit:
        BIT = int(args.bit)
        print("Using {} bit".format(BIT))

    num = (args.num)
    matchObj = re.match( r'0x(.*)|0X(.*)', num, re.M|re.I)

    if matchObj:
        # Hex input
        hexnum =  matchObj.group(1)
        decimalnum = toDec(hexnum)
        hexnum =  toHex(decimalnum) # To get the right padding
        binarynum = twocomplement(decimalnum, BIT)
    else:
        # Decimal input
        decimalnum = num
        hexnum =  toHex(decimalnum)
        binarynum = twocomplement(decimalnum, BIT)


    print("\tBase 10: {}".format(decimalnum))
    print("\tBase 16: {}".format(prettybinary(hexnum,2)))
    print("\t2's Compliment binary: {}".format(prettybinary(binarynum, 8)))

```
