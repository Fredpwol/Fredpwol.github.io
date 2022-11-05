---
title: Bit-wise operations and their use-cases
date: 2021-05-16 10:00:00 +07:00
tags: [programming, computer science]
description: deep dive into how and when to use bitwise operators
---

# Introduction

When learning how to write code we encounter so many operators like arithmetic operators and logical operators and for the most part, these operators solve most of our needs. but some operators are kept out of light not intentionally though they are called bitwise operators.

Today I would like to share more insight into these operations and what they are.

Some point to note is that this article isn't language-specific and no matter what language you code in you can still get a thing or two from it, I used python in this article to explain some concepts because it's the language I'm more comfortable with but that doesn't mean you won't get anything out of this article.

## What are bitwise operators?

To simply put it Bitwise operations are performed on data on the bit level or representation of the data. In classical programming, we can perform bitwise operations on numeric data types like Integers and floating-point numbers, although note not all languages like python, C/C++ supports bitwise operations on floating-point numbers, JavaScript however supports this.

Here are all the available bitwise operators and their symbolic representation.

<table>
<thead>
<tr>
<th>Operation</th>
<th>Sign</th>
</tr>
</thead>
<tbody>
<tr>
<td>and</td>
<td>&</td>
</tr>
<tr>
<td>or</td>
<td>|</td>
</tr>
<tr>
<td>not</td>
<td>~</td>
</tr>
<tr>
<td>xor</td>
<td>^</td>
</tr>
<tr>
<td>right shift</td>
<td>>></td>
</tr>
<tr>
<td>left shift</td>
<td><<</td>
</tr>
</tbody>
</table>

## bitwise and operator

The bitwise `and` operator behaves like the logical `and` operator with bits instead of booleans so say we have a bit with rep 1 and another with 0 when we perform the bitwise `and` operation we get 1 & 0 = 0, so the bitwise `and` operator returns 1 if both bits are set else 0 for example

```python
a = 3 #the binary representation of 3 is 0011
b = 5 #the binary representation of 5 is 0101

c = a & b
"""
if we perform the bitwise and operation on the two values we get.
0011
0101 &
----
0001 =>  which is equal to 1 in decimal so c will be equal to 1
"""
```

The bitwise `and` operator can be used for bit masking which is to make sure some bit is set while the rest are turned off. say we have the binary representation of 7 as `111` and we want to keep the `least significant bit` which is the first number from the right set, while we switch the remaining bit to 0. We perform the bitwise `and` on the number with 1 which is represented as 001 then we have

```
111
001 &
---
001
```

### Is Even

We can use a property of even and odd numbers in binary representation to check if a given number is even or odd. here's a list of the first 10 integers and their binary representation.

- 0 => 0
- 1 => 1
- 2 => 10
- 3 => 11
- 4 => 100
- 5 => 101
- 6 => 110
- 7 => 111
- 8 => 1000
- 9 => 1001

If you notice from above all even numbers least significant bit i.e (The first bit from right) is 0 while for odd numbers it's 1. with this we can be able to create a function that takes in an integer and return if it's even or odd, so our function would look like this.

```python
def is_even(num):
    if num & 1 == 1:
        return False
    return True
```

we check if our LSB (least significant bit) is set by bit masking it with the bitwise and operator if it's set we know it's odd and we return false else the number is even and we return true.

## bitwise or operator

The bitwise `or` operator is used to perform the `or` operation on a sequence of corresponding pair bits and return 1 if either of the pair of bits is set else 0.
for Example

```python
a = 5 # 0101
b = 7 # 0111

c = a | b

"""
0101
0111 |
----
0111
"""
```

As you can see the or operator creates a union of the two bits. This feature can be used for role allocation, but we'll come back to that later.

## bitwise not operator

The not operator returns a [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) of a bit it takes in a bit and flips the bit representation i.e given a binary number 010110 the not or twos complement of the number will 101001.

### Integers gotchas when using bitwise operators

So there are some things you need to be aware of when using bitwise operators. One is how many bits can be allocated to an integer and How are negative numbers represented by the computer? So Integers have a max bit size of 32 bits. That means the max value an integer can hold is 2^32, but that's not the case for positive integers because we need an extra bit to represent the sign of the number if it's positive or negative and we do that by setting the most significant bit i.e the first bit from the left to 0 if the number is positive and 1 if the number is negative. We have a bit size of (2^31)-1 as the maximum number a positive integer can have and -2^31 for negative integers. However, in python the size of the integer is allocated implicitly by the interpreter but for other languages like c/c++ running on a processor with a high bit size you can use the `big` keyword using large integers. 32 bits integers will be represented like this in binary.

```
5 => 00000000 00000000 00000000 00000101
-6 => 11111111 11111111 11111111 11111010
```

So with this knowledge, if we perform ~5 we get -6 returned ~3 = -4, and so on. the idea is that if we complement a positive integer we get a negative integer + 1. and if we perform the not operation on the negative number we get the original positive number.

There are some use-cases of the not operator, A trick that I can think of is for rounding down positive floating numbers. However, this operation can't be done in some programming languages like python which I use to explain this operator in this article. If you find yourself in a JavaScript environment Instead of using `Math.floor` method you can just use `~~floatingnum` to round your number like so.

```javascript
~~1.5021; // this will return 1
```

## bitwise xor operator

The bitwise returns 1 if either pair bits aren't the same and return 0 if pair bits are identical i.e

```
1 ^ 0 = 1
0 ^ 1 = 1
0 ^ 0 = 0
1 ^ 1 = 0
```

### Some properties of the xor operator

1. Performing the xor operation on its self will return 0 i.e ` x ^ x == 0`.
2. Performing the xor operation on a value with negative 1 will return the two's complement of the value or not `x ^ -1 == ~x`.
3. lastly xor of a value with 0 equals the value ` x ^ 0 == x`.

### Integer swapping

A very useful use case of the xor operator I discovered recently was using it to swap two variables without the need of a temp variable, pretty cool right? Here's the python code.

```python
x = 10
y = 15

x = x ^ y
y = x ^ y
x = x ^ y

print("x", x) # prints x 15
print("y", y) # prints y 10
```

Explaining this concept can be very tricky but bear with me as I try to explain it.
I showed you some bitwise properties earlier well with that knowledge we can be able to decode what's going on above.

```
the binary representation of 10 is 1010
and the binary representation of 15 is 1111

x = x ^ y => 1010 ^ 1111 => 0101
```

So now our new value for x is `0101`.

```
y = x ^ y => 0101 ^ 1111 => 1010
```

To further understand what happened above let's expand the expression like so.

```
y = (1010 ^ 1111) ^ 1111
```

since bitwise operations are [associative](https://www.mathwarehouse.com/dictionary/A-words/definition-of-associative-property.php#:~:text=Definition%3A%20The%20associative%20property%20states,where%20you%20put%20the%20parenthesis.) we can also write it as.

```
y = 1010 ^ (1111 ^ 1111)
```

from the properties above we know that `x ^ x = 0` so,

```
y = 1010 ^ 0
```

since x ^ 0 = x,

```
y = 1010
```

So from this, we can be able to know how we get the value of `x`.

```
x = 0101 ^ 1010 => (1111 ^ 1010) ^ 1010 =>
    1111 ^ (1010 ^ 1010) => 1111 ^ 0 => 1111
```

That was a lot to unpack, but if you were able to understand this is basically how to swap integers using bitwise operators.

## bitwise left-shift operator

The bitwise left-shift operator is used for moving bits n step to the left. So let's say we have a number represented in binary as 100110 if we perform 1 left-shift on the number like so 100110 << 1, we get 1001100. so what happened here is that all the bit from right to left shifted one position to the left and the LSB is replaced with 0. If we were to shift it by two we would end up with 10011000.

I mostly use them to get the power of two i.e a number that is 2 ^ n, because 2's power in binary always starts with 1 and followed by 0's, for example. 2 => 10, 4 => 100, 16 => 10000. so I'll just shift 1 two the power and I get the value. so 1 << 2 == 4 e.t.c.

### Converting RGB to HEX

One use-case of the left shit operator is converting a tuple of RGB color to its hexadecimal form so let's take a look at how it will be done.
The first thing to note is that for each channel in RGB tuple the value ranges from 0 - 255, which means that the can be represented as 8 bits integers. So given our 32-bit integer bit space, and an RGB tuple of (10, 128, 1) can be represented as.

```
00000000 00000000 00000000 00001010 => 10 => R
00000000 00000000 00000000 10000000 => 128 => G
00000000 00000000 00000000 00000001 => 1 => B
```

A hex color is a string of a hexadecimal value of length 6. The hex value is divided into 3 parts each represents either red, green, and blue. For example `#10ff0e` red = 10, green = ff, e = 0e in hex. So to combine RGB and produce a hex value we use left-shift to move every channel to its corresponding position. we shift the red channel to the 24th bit by shifting all its bit by 16 to the left then we do that also for the green but by 8 and we leave the blue channel the same. we then merge the integers by performing a bitwise or on them so it would be like so.

```
00000000 00001010 00000000 00000000
00000000 00000000 10000000 00000000
00000000 00000000 00000000 00000001 |
-----------------------------------
00000000 00001010 10000000 00000001 => output
```

So our python code will be like this.

```python
def rgb_to_hex(rgb):
    res = rgb[0] << 16 | rgb[1] << 8 | rgb[2]
    _hex = hex(res).replace("0x", "")
    return f"#{_hex}"
```

## bitwise right-shift operator

The bitwise right-shift operator behaves like the left-shift operator but instead of shifting the bits to the left by n, it shifts them to the right by n, therefore reducing the value. let's take a number with a binary representation of 101101 if we perform a right shift operation on the number with 1 shift we would end up with 10110 as our new value. we can also move it by any amount as we did when using left-shift.

### Converting HEX to RGB

This time we are trying to convert a hex string to RGB, we use the same concept from above but inverse but this time our input is a single value. We would shift our bits to the left and bitmask it using the bitwise `and` operator then get our value, so it will be something like this.

```
inp => 00000000 00001010 10000000 00000001

       00000000 00000000 00000000 00001010 >> 16
       00000000 00000000 00000000 11111111 & => R

       00000000 00000000 00000000 10000000 >> 8
       00000000 00000000 00000000 11111111 & => G

       00000000 00000000 00000000 00000001
       00000000 00000000 00000000 00000000 & => B
```

So this is how the conversion works in theory, here's the python code that performs this operation.

```python
def hex_to_rgb(hex_string):
    _hex = eval(f"0x{hex_string.replace('#','')}")
    r = _hex >> 16 & 0xff
    g = _hex >> 8 & 0xff
    b = _hex & 0xff

    return (r, g, b)
```

0xff here is used for bit masking its binary form is `11111111`.

## Creating Permissions and Role

Bitwise operators are most commonly used in applications that require users to have roles and access privileges. You'll most likely come across this if you work on an application that requires these features. So in this example, we would put everything together we've learned so far in this article to create a mini function that emulates user access role.

```python
class Permissions:
    CAN_READ = 1
    CAN_WRITE = 1 << 1
    CAN_CREATE = 1 << 2
    CAN_DELETE = 1 << 3
```

Here you can use the enum type if you come from a language that supports enums, python doesn't support this type that's why I used class in place of it. one thing to note here is all the numbers are 2 ^ n from 0 to 3, which means they are the power of 2.

```python
user = {
    "name": "sam",
    "role": Permissions.CAN_READ | Permissions.CAN_WRITE
    }
```

We create a user dictionary object with a name and role. the role is assigned by performing a bitwise `or` on the access we want to assign to the user. so if `CAN_READ` = `1` and `CAN_WRITE` = 10 performing a bitwise `or` will return `11`. To check if a user has access we make sure the nth bit is set i.e for `READ` we check the 0th bit from the right, `WRTIE` 1st bit, CREATE 2nd bit e.t.c.

```python
def do_operation(user, operation):
    if operation == "create":
        if Permissions.CAN_CREATE & user["role"] == \
         Permissions.CAN_CREATE:
            open("file", mode="w")
            print("file created successfully")
        else:
            print("Create Permission Required!")

    elif operation == "write":
        if Permissions.CAN_WRITE & user["role"] == \
        Permissions.CAN_WRITE:
            with open("file", "w") as f:
                f.write("00000000000000000000")
                print("wrote text to file!")
        else:
            print("Write Permission Required!")
    elif operation == "read":
        if Permissions.CAN_READ & user["role"] == \
         Permissions.CAN_READ:
            with open("file", "r") as f:
                print(f.read())
        else:
            print("Read Permission Required!")
    elif operation == "delete":
        if Permissions.CAN_DELETE & user["role"] == \
         Permissions.CAN_DELETE:
            os.remove("file")
        else:
            print("Delete Permission Required!")

```

We create a function `do_operation` that takes in a user dictionary and an operation a user should perform, here's a list of the operations the user can perform.

- create
- write
- read
- delete

Permission is required from the user to perform the operations. if the user doesn't have the right access privilege the operation will fail. we use the bitwise `and` operator here to check if the nth bit for the corresponding permission is set or not.

```python
do_operation(user, "create") #Create Permission Required!
do_operation(user, "write") # wrote text to file!
do_operation(user, "read") # 00000000000000000000
do_operation(user, "delete") # Delete Permission Required!
```

As you can see the operations that the user doesn't have access to failed while the rest were successful. you can play around with the code to discover new kinds of stuff and as a little assignment for you try finding out how to create a super admin role that has access to do everything.

## Conclusion

Well, that's mostly it for working with bitwise operations hope you learned a lot from this and made an impact in your coding style there is a lot you can do with bitwise operators, go online and search for them or play around with it and see what cool stuff you can discover.
