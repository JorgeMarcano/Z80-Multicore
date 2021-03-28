# Brainstorm

This document will hold a description of my brainstorming during this project. 
This means the document will likely be messy, and while I will try to include some form of table of content here at the beginning, there is no guarantee it will be accurate or precise.

---

### 28 March, 2021 @ 0:19
After having spent the week reading different datasheets and user manuals for the different main chips (Z80 and Z80 PIO), I think I have a fairly clear idea of the different features they offer.

Since I have never used a Z80 myself, I will likely start by building a classic, single Core Z80 circuit.
This will show me the ropes of the chips in a more practical manner.

The main challenge I believe will be the shared RAM between both Cores
I do however have a few possible ideas that might solve this issue:

1. Add an additional "Cache" to each Core that will take over for any write commands, save the data that will be written, and then, once the main RAM is available update said locations with the new memory. Both CPUs however will be able to read directly from the RAM to prevent the slowing down of read times. This will have the avantage of never having to worry about write speeds, since the Cores will essentially always have access to their respective Caches. The disadvantages is that the RAM will not always have the most up to date values, so if a value is to be shared between the two cores, there is no guarantee that the updated value will be available to be read from RAM by the other Core. Another disadvantage is that they cannot read at the same time, thus if they both need to read from RAM, one will have to wait for the other one.

1. This solution is very similar to the previous one except that we make the Cache bidirectional (essentially one cache for writes, one for reads). This will slow down the read times by adding a middle man that will require the cpu to request memory from the Cache as a proxy, then the cache will fetch from the RAM. This will allow for a faster read of both Cores attempt to read since they will be able to utilize the Wait pin instead of the BUSREQ. I don't know if the BUSREQ or the Wait is better to be honest, but at least, whenever the Core is attempting to read from RAM, they will be blocked off from the Bus altogether through some tri-state buffers. The RAM itself will also be isolated from the Bus, which will allow the Cores to talk to other peripheries at the same time as the other is trying to access RAM, which will allow for much faster cycles (better coordination).

1. Another solution would be to simply connect the two Cores directly to the RAM and have them simply use the BUSREQ and/or WAIT signals to coordinate between the reads and writes to RAM. While this one will likely take the least amount of external logic. It will also be the slowest, and quite frankly, probably too difficult to achieve.

For the two first solutions, the writes from the Caches to the RAMs will likely be done using some sort of counter. The Caches themselves will likely be some sort of RAM where they save the Data and the Addr Low and Addr High. For example, using a single RAM chip:

Core -> 3x 8-bit Register -> 3 consecutive Cache RAM locations -> RAM  
There are counters between the registers and the Cache RAm that will increment and keep the location after the newest value/addr combo  
There are also counters between the Cache RAM and the RAM keeping track of the next value/addr to write to the RAM  
If the second counter ever reaches the first counter, it will pause since it would have updated everything already.  
It does not matter if the counter overflow and restart since the logic will stay the same, unless if the first counter catches back up to the second one (it doesnt a full lap more than the second), the it should place the Core in a WAIT state to give time for the Cache to write to RAM, other the information will be lost.

For the second solution, the Read Proxy does not require much more than a single 8-bit register for the data in one direction (RAM -> Core) and a single 16-bit register for the addr, also in one direction (Core -> RAM), along with some logic.

---