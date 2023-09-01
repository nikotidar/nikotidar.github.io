---
layout: post
title: "An Idiot's Guide to Bruteforcing [PART I]"
author: "rndtx"
---

The other day i found a password-protected zip file on an old hard drive. Seeing that it had an intruiging name, and that i didn't know the password, i told my self that this would be a good challange to learn bruteforcing.

_Bruteforcing_. The act of systematically checking every possible password until the correct one is found - [see Wikipedia for more.](https://en.wikipedia.org/wiki/Brute-force_attack)

---

# The Math Behind It
I know, now you'll start to hate me. It seems that the definition of bruteforcing requires you not to worry about math. If anything, bruteforcers want to get away from math. But once you get started with it, it's not possible to evade it.

So i told myself i needed to learn it, at least to be able to know how long cracking this file would work.

Bruteforcing is pure math and i didn't understand that in the beginning. Which is why i ended up frustrated after trying to crack on password 24/7 for 30 consecutive days with no results :(.

The math it is simple. Let's define its two component and then their relationship:
1. Character set
2. Password length

# Character Set
The character set is a list of characters the password, you are trying to bruteforce, contains. Typically it's made up of a combination of the following types:
* lowercase alphabet [a-z]
* uppercase alphabet [A-Z]
* numbers [0-9]
* extra characters [.-!]

Let's use a practical example. In the case of the file i wanted to crack, i used the following command to bruteforce it.

```sh
$ fcrackzip -b -c aA1! -u -l 4-10 file.zip > result.txt
```
In ```fcrackzip``` the ```c aA1!``` parameter defines the character set. In this case it means that we are going to use lowercase and uppercase alphabet numbers and extra characters.

```sh
Character set = a-z + A-Z + 0-9 + .-!

# of characters = 26 + 26 + 10 + 33 = 95
```
If you add all these characters, you will end up with a total of 95. Let's go look at password length.

# Password Length
It defines how many characters the password, we are trying to crack, has.

When bruteforcing, the password length tends to be an educated guess. It's rare to exactly know how long the password is. More often we know a minimal and maximum password length.

If we took at the command from above again, we can see that i, too, made an educated guess concercing the password length.

```sh
$ fcrackzip -b -c aA1! -u -l 4-10 file.zip > result.txt
```
I estimated it to be between 4 and 10 characters.

# The Relationship
Now that we have the basic information straightened out, let's see how this helps us. Let us calculate the total number of possible combinations.

So far, we have established the following:

```sh
Minimum password length : 4
Maximum password length : 10
Size of character set : 95
```
The relationship between the two is described by this equation:

![Character Set](https://i.imgur.com/FPip4mE.png)

For each character in the password you have 95 possibilities.

```sh
95 x 95 x 95 x 95 = 81,450,625 possible combinations
```
That means for a 4 characters password, you have 81 million possible combinations. For a password with 5 characters the total number of combinations is:

```sh
95 x 95 x 95 x 95 x 95 = 7,737,809,375 possible combinations
```
We can obviously also calculate the number of combinations for a 6, 7, 8, 9 or 10 character password - if it interests you, do go ahead and calculate it.
