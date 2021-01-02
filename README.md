
# Quake III Algorithm (Fast Inverse square root)
 - How to approximate an inverse of a square root
 
### Step 1

- The function ``y.view(np.int32)`` constructs a view of **Y**’s memory with a **int32** as the new data-type. This can cause a reinterpretation of the bytes of memory.

- Basically it will let work with the mantissa of a float number

```C
    // Code in C
    i = * ( long * ) &y
```

*****
------


Given two numbers **M** and **E**, one being the mantissa (M) and one being the exponent (E)

$$
\begin{align*}
M & & 01001110010000000000000\\
E & & 10001001\\
\end{align*}
$$

It is possible to get the bit representation as $ 2^{23} * E + M $, because $2^{23} * E$ just shifts E by 23 digits.

To get the actual number behind the bits use the formula $ (1 + \frac{M}{2^{23}}) * 2^{E - 127} $

----

- Taking the logarithm base 2 of the expression $ (1 + \frac{M}{2^{23}}) * 2^{E - 127} $

$$
\begin{align*}
&\log_2((1 + \frac{M}{2^{23}}) * 2^{E - 127}) \\
&\log_2((1 + \frac{M}{2^{23}})) + \log_2(2^{E - 127})\\
&\log_2((1 + \frac{M}{2^{23}})) + E - 127 \\
\text{Using} \log_2(1+x) \approx x + \mu &\\
&\frac{M}{2^{23}} + \mu + E - 127 \\
&\frac{M}{2^{23}} + \frac{2^{23} * E}{2^{23}} + \mu - 127 \\
&\frac{1}{2^{23}} * ({M + 2^{23} * E}) + \mu - 127 \\
\end{align*}
$$

${M + 2^{23} * E}$ is bit representation of a number, in some sense the bit representation of a number is its own logarithm

#### Step 2 

> ##### Remider
> 
> Shifting a number to the left doubles it
>> x << 1 
>>
>> 110 = 6
>>
>> 1100 = 12
>
> Shifting a number to the right halfs it
>> x >> 1 
>>
>> 110 = 6
>>
>> 11 = 3

- So if it is done to the exponent:
 - Doubling an exponent squares the number $x^1 => x^2$
 - Halfing the exponent gives an square root $x^1 => x^{1/2} = \sqrt{x}$
 - Negating the exponent will give 1 divided by the number $ x^{1/2} => x^{-1/2} = \frac{1}{\sqrt{x}}$
 
 ------------
 ------------
 
 In some sense, it is stored in $i$ the logarithm of $y$ up to some scaling and shifting. Then the problem becomes easier work with the log of y.
 $$
 \begin{align*}
 y &= \frac{1}{\sqrt{x}}\\
 \log_2(y) &= \log_2(\frac{1}{\sqrt{x}})\\
 & = \log_2(y^{-\frac{1}{2}})\\
 & = -\frac{1}{2} * \log_2(y)\\
 \end{align*}
 $$
 The code ```np.int32(i >> 1)``` represents this division by 2 of the log of Y, sinse the $i$ represents the logarithm of Y, itself
 
 ------
 ------
 
 #### Explanation of ```np.int32(0x5f3759df)```
 
 $$
 \begin{align*}
 &\text{Let } \Gamma \text{ be the solution} \frac{1}{\sqrt{y}} \text{ then:}\\
 &\log(\Gamma) = \log(\frac{1}{\sqrt{y}})= -\frac{1}{2}\log(y)\\
 &\text{Replacing the logarithm with the bit representation } \frac{1}{2^{23}} * ({M + 2^{23} * E}) + \mu - 127 \\
 &\log(\frac{1}{2^{23}} * ({M_\Gamma + 2^{23} * E_\Gamma}) + \mu - 127) = -\frac{1}{2}\log(\frac{1}{2^{23}} * ({M_y + 2^{23} * E_y}) + \mu - 127)\\
 \\
 &\text{By doing a typical as it is easy to see, the result is:}\\
 \\
 &({M_\Gamma + 2^{23} * E_\Gamma}) = \frac{3}{2} * 2^{23} * (127 - \mu) - \frac{1}{2}* ({M_y + 2^{23} * E_y})\\
 \end{align*}
 $$
 
 Then, it is possible to see that ```np.int32(0x5f3759df)``` = $\frac{3}{2} * 2^{23} * (127 - \mu)$
