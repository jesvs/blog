---
title: Speed up badblocks check
author: Jesús Sánchez
type: post
date: 2019-01-15T23:29:03+00:00
url: /speed-up-badblocks-check/
categories:
  - Tecnología

---
Having ordered an external hard drive from China… let me rephrase that; having ordered a [really cheap hard drive][1] from an unknown source at the farthest side of the world, getting it handled by the not so delicate hands of the Mexican post office, and getting it delivered to my door by bicycle, of course I had to check it thoroughly before putting any data in it.

I started with the well known way of checking for bad blocks on a storage device:

<pre class="wp-block-preformatted">badblocks /dev/sdX</pre>

This command by default checks for bad blocks only by reading. If you want to also check writing non-destructively, the -n option must be provided.

<pre class="wp-block-preformatted">badblocks -n /dev/sdX</pre>

This was what I originally tried, but after 2 hours it had only checked 3% of the entire drive, not acceptable! There has to be a way to improve the performance.

### How to speed up badblocks read/write check

First, I recommend you get your hard drive&#8217;s block size, I&#8217;m not sure if this impacts performance but it&#8217;s more reliable:

<pre class="wp-block-preformatted">lsblk -o NAME,PHY-SEC /dev/sdX<br />NAME   PHY-SEC<br /> sda        <strong>512</strong></pre>

In case your hard drive is empty or you don&#8217;t mind losing its data, use the **w**rite mode option (_-w_). This mode is faster than the -n option but the entire drive contents will be overwritten.

Finally, choose a bigger number of blocks to test at a time using the -c option; the default is 64. By benchmarking I found that 128 was a bit faster but YMMV. The badblocks command accepts a [last block] option, this way you can benchmark different -c values:

<pre class="wp-block-preformatted">time badblocks -b512 -c128 /dev/sdX 1000000</pre>

If you want badblocks to show the scan progress, provide the -s option. This is what the final command looks like:

<pre class="wp-block-preformatted">badblocks -b512 -c128 -w -s /dev/sdX</pre>

BTW. The drive was 100% free of errors.

 [1]: https://www.aliexpress.com/item/Blueendless-External-Hard-Drive-1TB-2TB-500GB-USB-3-0-Hard-Disk-HDD-2-5-HD/32880340173.html