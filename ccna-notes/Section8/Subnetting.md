# CIDR Classless Inter-Domain Routing

![alt text](image.png)

![alt text](image-1.png)

![alt text](image-2.png)

# Subnetting Overview 

![alt text](image-3.png)

![alt text](image-4.png)

![alt text](image-5.png)

![alt text](image-6.png)

# Subnetting Class C Networks and VLSM

 ![alt text](image-7.png)

 ![alt text](image-8.png)

 ![alt text](image-9.png)

 ![alt text](image-10.png)

 ![alt text](image-11.png)

 ![alt text](image-12.png)

 ![alt text](image-13.png)

 ![alt text](image-14.png)

 ![alt text](image-15.png)

 ![alt text](image-16.png)

 # Subnetting Practice questions

 ![alt text](image-17.png)

 ![alt text](image-18.png)

 ![alt text](image-19.png)

 # Variable Length Subnet Masking 

![alt text](image-20.png)

![alt text](image-21.png)

![alt text](image-22.png)

Example: 

![alt text](image-23.png)

![alt text](image-24.png)

Explanation: 

- The starting point

You're given the block 200.15.10.0/24 — that's 256 addresses (200.15.10.0 through 200.15.10.255), and it's yours to divide up between two offices: New York and Boston.

Why /27?

A /24 means the first 24 bits are the "network" part, leaving 8 bits for hosts (256 addresses). But you don't need 256 addresses per office — you just need to split the block in two.

By borrowing 3 more bits (going from /24 to /27), you shrink each subnet down to 32 addresses (2^5 = 32, since 5 bits are left for hosts). Out of those 32:

1 address = network address (unusable, it just names the subnet)
1 address = broadcast address (unusable, it's for talking to everyone at once)
30 addresses left over for actual devices (hosts)

That's where "/27 supports 30 hosts" comes from.

The bit chart (why /27 looks the way it does)

That row of numbers (128, 64, 32, 16, 8, 4, 2, 1) is just how one octet (8 bits) is built in binary. A /27 means: take the first 27 bits across the 4 octets and lock them as "network," leave the remaining 5 bits (in the last octet) as "host."

That's why the red line is drawn right after the "32" column in the last octet — everything to the left of it (128+64+32 = the top 3 bits) is fixed/network, everything to the right (16+8+4+2+1) is free for host addresses.
gdfgdfg
Splitting into the two offices

Since each /27 chunk is 32 addresses wide, you just count in blocks of 32 starting from 200.15.10.0:

Office	Network Address	Usable Hosts	Broadcast
New York	200.15.10.0/27	.1 to .30	.31
Boston	200.15.10.32/27	.33 to .62	.63

New York gets the first 32-address block (0–31), Boston gets the next 32-address block (32–63). Simple as that — just chopping the original /24 into equal 32-address slices and handing one to each office.


- Calculations

Step 1: Figure out how many subnets you need

You have 2 offices (New York, Boston), so you need at least 2 subnets.

Step 2: Figure out how many bits to borrow

Bits borrowed determine how many subnets you get, using 2^n:

Borrow 1 bit → 2^1 = 2 subnets ✅ that's enough for our 2 offices

So we borrow 1 bit from the host portion. That bit comes from the /24, turning it into a /25... but wait, our answer used /27, borrowing 3 bits. Why?

Because it's not just "how many subnets" — it's also "how many hosts do I need per subnet." If you only needed 2 subnets with no host limit, /25 would work. But this design decided 30 hosts per office was the target (maybe for future growth/planning), and 30 hosts requires 5 host bits — which forces you to /27. So really the driver here was hosts needed, not just subnet count. In practice you check both and pick whichever borrowing satisfies your requirement.

Step 3: Do the math for "how many host bits do I need"

Formula: 2^h − 2 ≥ hosts needed (the −2 accounts for network + broadcast address)

You need ~30 hosts:

2^5 − 2 = 32 − 2 = 30 ✅ exactly enough

So you need 5 host bits left over.

Step 4: Convert that into your subnet mask

An IPv4 address has 32 bits total. If 5 bits must stay as "host bits," the rest are network bits:

32 − 5 = 27 → that's your new prefix: /27

Step 5: Find your "block size" (the jump between subnets)

Block size = 256 ÷ (number of subnets your borrowed bits create) — or more simply, block size = value of the lowest bit you borrowed.

With /27, the last host-bit column is "32" (128, 64, 32 | 16, 8, 4, 2, 1 — red line right after 32). So your block size is 32.

That means every subnet is 32 addresses wide, and they start at multiples of 32:

0, 32, 64, 96, 128, 160, 192, 224

Step 6: List out the subnets

Subnet	Network Address	Usable Range	Broadcast
1	.0	.1–.30	.31
2	.32	.33–.62	.63
3	.64	.65–.94	.95
...	...	...	...

You only needed 2 (New York = .0, Boston = .32), but the /27 mask actually gives you 8 total subnets (256÷32) — the other 6 are just unused/reserved for future offices.

The quick mental shortcut (once you get comfortable):

1. How many hosts do you need? → round up to nearest 2^h − 2
2. That tells you host bits → 32 minus that = your prefix
3. Block size = 256 ÷ 2^(borrowed bits), or just "the value of the last borrowed bit"
4. Count up by block size to list your subnets

# More Examples

![alt text](image-25.png)

![alt text](image-26.png)

![alt text](image-27.png)

![alt text](image-28.png) 

# Point to point subnetting example

![alt text](image-29.png)

# Subnetting Large Networks

![alt text](image-30.png)

![alt text](image-31.png)

![alt text](image-32.png)

![alt text](image-33.png)

![alt text](image-34.png)

![alt text](image-35.png)

![alt text](image-36.png)

# Subnetting Large Network Part 2

![alt text](image-37.png)

![alt text](image-38.png)

![alt text](image-39.png)

![alt text](image-40.png)

![alt text](image-41.png)

![alt text](image-42.png)

# Private IP Addresses

![alt text](image-43.png)

![alt text](image-44.png)

![alt text](image-45.png)

![alt text](image-46.png)

![alt text](image-47.png)

![alt text](image-48.png)

![alt text](image-49.png)

![alt text](image-50.png)

![alt text](image-51.png)











 

 









