---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/down-under/love-granni-e/","tags":["downunder_ctf_25","osint"],"dg-note-properties":{"tags":["downunder_ctf_25","osint"]}}
---

We are given an image, sent to us by our "grannie" and tasked with finding the old movie theater she used to go to.


>The image depicts the Epping Bridge in Sydney, Australia, likely in an earlier period given the attire of the people and the bridge's structure.
- *from google reverse image search*

![Pasted image 20250817103507.png](/img/user/img/Pasted%20image%2020250817103507.png)

Find the old epping bridge station:
![Pasted image 20250817103548.png](/img/user/img/Pasted%20image%2020250817103548.png)![Pasted image 20250817103600.png](/img/user/img/Pasted%20image%2020250817103600.png)
- Looks like the same platform to me

Now look for old movie theaters nearby:
![Pasted image 20250817103634.png](/img/user/img/Pasted%20image%2020250817103634.png)

first link:
![Pasted image 20250817103812.png](/img/user/img/Pasted%20image%2020250817103812.png)
- The Cambria Theatre was opened on 6th November 1915

Grannie mentioned that movies in her day were silent and the first talkie came out in the late 20s, so the name would have to be the `Cambria Theatre`, rather than `Epping Kings Theatre`.
I have an address too: `46 Beecroft Road, Sydney, NSW 2121`, which happens to be right next to the station as Grannie remembered:

![Pasted image 20250817104044.png](/img/user/img/Pasted%20image%2020250817104044.png)

however, this does not look like that building, and trying this address of the theater for the flag is incorrect.

![Pasted image 20250817104124.png](/img/user/img/Pasted%20image%2020250817104124.png)

The hint mentions "Sometimes old records get out of date, you might need to try the street number next door"

![Pasted image 20250817104157.png](/img/user/img/Pasted%20image%2020250817104157.png)

![Pasted image 20250817104208.png](/img/user/img/Pasted%20image%2020250817104208.png)
- looks like it goes to the back of the building

![Pasted image 20250817104235.png](/img/user/img/Pasted%20image%2020250817104235.png)
- yep, that looks like the theatre all right

this is also confirmed in the second link from our [search for nearby old theaters above](https://www.flickr.com/photos/29029178@N03/7221299438/in/photostream/)
So the flag should be `DUCTF{CambriaTheatre_47BeecroftRd_Epping}`
- NOT 46 Beecroft