---
title: fsck’d a full partition
author: Jesús Sánchez
type: post
date: 2019-01-02T10:52:09+00:00
url: /fsckd-a-full-partition/
featured_image: /wp-content/uploads/2019/01/IMG_5235.jpg
categories:
  - Tecnología
tags:
  - ext4
  - f2fs
  - fsck
  - kodi
  - raspberry

---
I run Kodi on my Raspberry Pi 3 mainly to watch live streams and the occasional YouTube video. Along with the media player I also run a pair of Go programs to scrape, store and analyze web data. On New Year&#8217;s Eve the root partition on a 32GB SD card got full, this rendered the whole system unresponsive. No big deal, I said. Without ssh access nor physical console, I pulled the power plug from the Pi, took the card out and mounted it on my workstation. Everything looked OK, I freed some space, unmount the partition, check the file system consistency… If by now you haven&#8217;t spotted what step I missed, keep reading, it could save you a lot of trouble.

So I mounted the SD card&#8217;s partition on my main PC, removed some temp files and old videos, 5 minutes after: 6GB left. Then I ran fsck.ext4 with auto-fix on that partition and… whoops, it reported an incredible amount of errors, the last one was about not being able to fix who-knows-what. “What the heck” I said, let&#8217;s run fsck.ext4 again on it. It still reported tons of errors, but this time it finished successfully. Big smile on my face, let&#8217;s see how looks.

Well, it didn&#8217;t look good, no good at all. The only directories left on the partition where /bin and /lost+found, inside where several files with strange ownership and permissions. That&#8217;s when I knew I had fsck&#8217;d up the partition. I spent the first hours of 2019 reinstalling Arch on a Raspberry Pi, how cool is that?

So I learned my lesson. If the root partition (or any other partition) gets full (as in 0 bytes left), and the system freezes or is unresponsive so that the last resort is to power it down, DO NOT RUN _fsck_ ON IT UNLESS IT IS BACKED UP! Treat this partition as read only –for now– and back up whatever you can from it, then run the file system check on it. If it reports no errors, you&#8217;re golden, otherwise you should reformat it and put back whatever files you backed up.

This time instead of using good ol&#8217; ext4 I&#8217;ll give Samsung&#8217;s f2fs a try. Hopefully it&#8217;ll make it through this 2019 without a hitch.

Luckily I had a week old backup so nothing of importance got lost, the Pi was up and running after 45 minutes, with the only trouble being Kodi not playing video, but it turned out to be just a matter of making my user a member of the **video** group. I&#8217;ll leave the error from the log here so it serves as reference for future me or some other poor sob.

<pre class="wp-block-preformatted">ERROR: CMMALPool::GetBuffer - failed pool:0x5a9711c0 omvb:(nil) mmal:0x5a983248 timeout:500<br />ERROR: CDecoder::FFGetBuffer Failed to allocated buffer in time</pre>

So, that&#8217;s what my 2018 transition to 2019 went, very exiting indeed. I wish you all a whole year of trouble and the strength to overcome it.