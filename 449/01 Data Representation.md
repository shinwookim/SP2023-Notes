# Data Representation

## Binary Encoding
The numbers we use are written **positionally**; the position of a digit within the numbers have meaning.$$2022=2000+000+20+2=2\times10^3+0\times10^3+2\times10^1+2\times10^0$$Typically, in everyday-life, we use a **base-10** number system in which any number is represented by digits $d_{n-1}\ldots d_1d_0$ where $d \in \{0, 1, 2, 3, 4, 5, 6, 7, 8, 9\}$. This number has the value $d_{n-1}×10^{n-1}+\ldots+d_1×10^1+d_0×10^0$. Notice that using $n$ digits, we can represent at most $10^n$ different values, with the smallest non-negative representable value being $0$ and the largest being $10^n-1$.

Unlike us, however, computers store and process information represented as two-valued signals because they can be readily be represented, stored, or transmitted. This **base-2** system is called **Binary**. The binary number system represents numbers similar to the base-10 system, but with digits  $d \in \{0, 1\}$. With this reduced symbol set, our representation now has the value $d_{n-1}×2^{n-1}+\ldots+d_1×2^1+d_0×2^0$. Hence, with $n$ digits we can represent at most $2^n$ different values, with the smallest non-negative representable value being $0$ and the largest being $2^n-1$.

We call each digit a **binary digit**, or a **bit**, and is written with a *lowercase* **b**. A group of 8 bits is called a **byte** and is written as an *uppercase* **B**. A **nibble** (or nybble) is a group of 4 bits. And lastly, a **word** refers to the *most comfortable number* of bits for a given CPU. For example, a 32-bit CPU has a word size of 32 bits, meaning we can do arithmetic with two 32-bit numbers at once. Note that with some operating systems (mainly x86 Windows), the meaning of **word** may be ambiguous as a **word** means 16 bits, and a **double word** means 32 bits on those systems.

Again, everything on a computer is represented in binary. Strings are just numbers encoded into symbols in a scheme called **UTF-16** or **ASCII**.

### Hexadecimal Number System
Since binary numbers only have two symbols to work with, their representation gets really long, really fast ($3,927,664_{10} = 11\ 1011\ 1110\ 1110\ 0111\ 0000_2$) and nice *round* numbers in binary look arbitrary in decimal ($1000000000000000_2 = 32,768_{10}$) since $10$ is not a power of $2$. To make it easier to read (for humans), let's choose a larger base that can still be easily converted to base-2. Our first choice of base is $4$, but base-4 is not much conciser than binary; larger bases such as base-32 requires 32 symbols, which isn't helpful (for us). Thus, the logical choice for the base is **base-8**, and **base-16**.

Historically, base-8 was commonly used in computer science, but with computers becoming faster and more powerful, the most common choice of a base in the modern day is **base-16** (also called the **hexadecimal number system**).

Base-16 represents values similar to the binary representation, but the digits now include 0-9 and A-F (with A-F representing decimal numbers 10 to 15). With this expanded symbol set, our representation has the values $d_{n-1}\times 16^{n-1}+\ldots+d_1\times16^1+d_0\times 16^0$. Hence, with $n$ digits we can represent at most $16^n$ different values, with the smallest non-negative representable value being $0$ and the largest being $16^n-1$.

Since 16 is a power of 2, it is very easy to convert base-2 to base-16. To do so, combine the binary representation into groups of 4 bits (moving from the left to right) and add padding to the right most group (if it is not a group of 4). Then, it is simply a matter of converting each group into a hexadecimal digit. To do the reverse (base-16 to base-2), it's simply a matter of converting each hexadecimal digit to a group of 4 bits (removing the padding in the right most group).

## Signed Numbers
### Overflow
See Data Rep II