<style>
    .centered {
        text-align: center;
    }

    .header {
        font-weight: bold;
    }

    .warning {
        color: red;
        font-weight: bold;
    }

</style>

<h1 class="centered header">Project 2</h1>
<p class="centered header">LC-3 ML</p>
<p class="centered header">Ahmed Muhammad</p>
<p class="centered header">2021-04-10</p>
<br>

<h2 class="centered header">Description</h2>
<p>
In this project we used all three of the LC-3's memory addressing modes. The LC-3 machine has to be loaded with 5 ASCII decimal characters starting from address x3000. The program will read in these values, extract the decimal value, add them all up, and store the result in memory address x20FF.
</p>

<br>
<h2 class="centered header">Code Walkthrough</h2>

<h3 class="centered warning">Word of Warning</h3>
<p>I tested this code in the online LC-3 simulator. Once the code is loaded at address x3100, the user has to keep hitting the "Step" button until they reach the address x3125. If the user hits the "Run" button then the desired output will not be generated. The simulator starts executing instructions that I did not load into it, I can't figure out why.  
</p>

<h3 class="centered header">The Beginning</h3>
<p>
The program starts at memory address x3100. We LEA register 7 with the address x3001 and subtract one from R7 so that R7 = x3000. From there we can LDR R0 - R4 with the contents in memory addresses x3000-x3004. The next few lines of code will AND the newly packed registers with "0000 0000 0000 1111" to clear out the ASCII information and extract the decimal value. From there each of these registers' value is added into R5. Now comes the fun part.</p>
<h3 class="centered header">The Fun Part</h3>
<p>
We could have ended the program rather simply, but so far we've only used LEA and LDR. To make use of ST and STI opcodes, we had to get a little tricky. The general idea that we came up with goes like so:

 <ol>
   <li>Populate a register with the final address (x20FF)</li>
   <li>Then use ST to store that address into some memory address</li>
   <li>Finally, STI to that memory address to fill the final address with the calculated sum.</li>
 </ol>

The tricky part was to add 20FF into a register... I ended up "looping" to get it done. Now that R5 had the final value in it, I figured we could reuse the first 5 registers. So I cleared R0, then populated it with the number 15. I cleared R1 and populated it with the number 17 (with two ADD instructions, of course). Then I cleared R2, and I kept adding R2 with R1 (17) and decrementing R0 (15) until R0 hit 0. In other words, I added R2 with 17 a total of 15 times until it had the decimal value of 255, to which I added a 1, and I was finally left with the hexadecimal value x0100 in R2.I negated that value to a negative x0100 in R2.  In python this would have looked like:

```python

r1 = 17; r2 = 0
for r0 in range(15):
    r2 += r1

r2 += 1     # r2 = 256 or x100
r2 *= -1    # r2 = -256
```

Now, if you remember, we had x3000 stored into R7. So this time, I stored R0 with 15, and R1 with x3000 from R7. As I decremented R0, I kept adding R1 with R2 (-x0100). When R0 became zero, R1 was left with x2100, from which I subtracted 1. R1 was left with x20FF, our destination for the calculated value. In javascript this would have looked like:
```javascript

let r7 = 12288;     // x3000
const r2 = -256;    // -x100
const r0 = [ ...Array(15).keys() ];

let r1 = r0.map( () => r7 += r2 ).pop()  // R1 = x2100

r1 -= 1             // r1 = x20FF

```

I ST'd x20FF into x30FF, and finally STI'd R5 (the calculated value) into x30FF which sent our final value to its final destination.
</p>

<h3 class="centered header">The End</h3>
