---
layout: post
title:  "Galois Fields"
date:   2024-01-16 -0500
categories: math cryptography
---

Many cryptography algorithms such as [AES-128](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) and [GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode) use the Galois field $GF(2^k)$. It's interesting to me that abstract algebra can have applications such as this, but I'm curious why it's useful. I couldn't find much information about this [online]((https://www.reddit.com/r/cryptography/comments/f4rn3o/why_use_galois_fields/)), but it turns out to a be an extremely natural choice.

As a reminder, a [field](https://en.wikipedia.org/wiki/Field_(mathematics)) is a structure in algebra that has addition, subtraction, multiplication, and division. You may want to encode the $2^{64}$ possible $64$-bit sequences as elements in your field. Then, you'd need a field with at least $2^{64}$ elements. $GF(2^{64})$ happens to be a field with exactly that many elements, and it's a theorem that field finites are unique (up to isomorphism), so it's our only choice. Thus, most basic reason to the Galois field $GF(2^k)$ is that we have a one to one map between sequences of $k$-bits and elements of our field--no wasted bits. 

Note: The reader may wonder why we need a field. For many algorithms, a ring (which is like a field but without division) may suffice. While using rings may be possible, in a ring multiplication is not invertible, so decoding our cyphers may be problematic. Even for authentication, not having invertibility would potentially make the algorithm more susceptible to birthday attacks.

How can we encode the elements of $GF(2^k)$? Each element can be representated as polynomials of degree $k-1$ with coefficients in $\mathbb Z/2 \mathbb Z$ (since $GF(2^k)$ is a field extension of $GF(2) = \mathbb Z/2\mathbb Z$). Thus, bit #$i$ can represent the coefficient $a_i$ of the polynomial $\sum_{i=0}^{k-1} a_ix^i$. That means that addition of two elements of $GF(2^k)$ is just adding these polynomials, and since we are taking everything mod $2$, this is just bitwise XOR. For me, this is the most compelling argument to use $GF(2^k)$–that we have such a simple encoding with efficient addition. Even another Galois field such as $GF(3^k)$ would not have this.

The other important operation is multiplication; we need to make sure that is efficient as well. This multiplication in fact is efficient, but for those who are interested we describe the algorithm. In order to specify a representation of $GF(2^k)$, we also need to fix an irreducible polynomial (cannot be factored mod $2$) of degree $k$, call it $p(x)$. If you choose different polynomials, you get different representations of the field, but we noted before that these are all isomorphic so you can just choose any. For two elements $a,b\in GF(2^k)$, remember we view them as polynomials. The product $a\cdot b$ is the remainder when you multiply these two polynomials and divide by $p(x)$. Since $p(x)$ is degree $k$, the remainder of this division is a degree $k-1$ polynomial, so it is an element of $GF(2^k)$. The following example, taken from this [paper](https://csrc.nist.rip/groups/ST/toolkit/BCM/documents/proposedmodes/gcm/gcm-spec.pdf), is the algorithm for $k=128$ fixing the polynomial $p(x) = 1+x+x^2+x^7+x^{128}$. This algorithm generalizes for all values of $k$ and is $O(k^2)$.

**Algorithm**
Each variable below is a sequence of $128$ bits. The leftmost bit is $a_0$ and the rightmost is $a_{127}$. We fix a special value $R$ which is $11100001$ followed by one hundred twenty $0$'s. This algorithm computes the value $Z = X\cdot Y$, where $X,Y\in GF(2^{128})$.  
```python
def gf_multiply(X, Y):
  Z = 0
  V = X
  for i in range(0, 127):
    if Y[i]==1:
      Z = Z + V
    if V[127] == 0:
      V = rightshift(V)
    else:
      V = rightshift(V) + R
  return Z
```
