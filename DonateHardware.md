

# Hardware for Memcached Project #

Hey, Dormando here. I do maintenance work on memcached. Usually I queue up
work and run tests on borrowed hardware here and there. It'd be pretty darn
spiffy if I could have access to hardware (either at my house or in some colo
somewhere) to run tests on anytime.

There are lots of alternatives these days, but memcached sorta runs half the
internet. I've spent a lot of spare time and honestly a lot of my own money on
the project. I want to see if one (or more) people would like to stand
up and give back.

If you want to contribute, skip to the bottom.


---

# Why do you need hardware? #

I can get pretty far with my desktop, but if we want to find the same
bottlenecks you'll see in production, we need to push to the breaking point.

## Version over Version verification ##

I would love to develop a better performance regression suite, and
be able to run this across versions to ensure performance gets better and not
worse. Having ready access to hardware means I can kick off regression tests
against patches without having to schedule or scruitinize them with great
effort.

Right now, we have a general idea that things don't get slower, but it's very
hard for me to verify without hopping on someone's box and running tests.

## Vertical Scalability Improvements ##

It's pretty easy to get past 100,000 requests per second per instance. Out of
the box and on decent hardware memcached can even do 400k+. As memory prices
fall, cores increase, and we potentially back memory with SSD's, the number of
users hitting these limits will increase. To ensure future proofing, and that
larger and larger installations can make use of memcached, we need a way to
verify improvements.

## Latency Stability ##

Another big deal to people is latency of requests, especially under load and
over the network. I want to run tests to run up performance until latency
falters, then work on both scaling the total throughput (for batch
processing), and the most amount of work doable without impacting latency.

As is things are pretty good so long as you're not swapping. We want to
improve this (but you still can't swap).


---

# What type of hardware are we looking at? #

## Minimum ##

At minimum, a fast 16 core machine with 8g of RAM, and two 1gbit NIC's.
Don't have to be the fastest cores on the market, but very slow cores don't
help much.

High speed NIC's (10gbps) are desireable but not entirely necessary. It's
possible to test with smaller packet sizes to get most of what we need.

Box must have at least two NUMA zones. Machines with many cores and many CPU's
are split into multiple memory areas, and performance is often extremely
dependent on getting the NUMA fiddling correct; with such a machine we can
make memcached NUMA aware and ensure it stays that way.

## Bonus #1 ##

Two of them. Even if both aren't up to the total specs as noted below, having
a second machine to generate load with a large number of cores would be
fantastic. Otherwise I'll see how far I can get with my quadcore i7, or spring
for some extra load generator hardware with my own money.

More than two is probably unnecessary.

## Bonus #2 ##

Even more cores! 24 or 32.
I can easily get memcached to scale to about 12 cores today, so
fewer than 16 won't get us very far into the future.

## Bonus #3 ##

10gbps NIC's. This is probably only useful if we get two machines, as I
definitely don't have one of these cards and they are very expensive. It would
be a nice to have as we can work on scaling the network usage to 10gbps or
more.

## Bonus #4 ##

SSD's. One or more (ideally at least two) intel 320 300G SSD's. Memcached
relies solely on RAM, but these days we should be able to write engines which
utilize the "slower" SSD memory to potentially greatly expand the amount of
data you can cache per node, or to persist between restarts. (not necessary
for a secondary load generating box)

## Bonus #5 ##

Gobs of RAM (also not necessary for a secondary load generator box). Load up
the box with as much RAM as you're willing to give. This plus NUMA will allow
us to do more specific tests on scaling data structures, fill rates,
compression, etc.

Not terribly necessary, so I'd rather have any of the above bonus notes before
extra RAM beyond the minimum of 8G-ish.


---

# What do We The People get from this? #

I will continue to chuck a few days or weeks per month toward maintaining
memcached, and hopefully bringing it kicking and screaming into the future of
modern server hardware. I do this part for free, though donations or free
lunches are nice :)

I will be able to make memcached NUMA aware, ensure 10gbps scalability, expand
its memory limits into SSD's, and improve its online documentation. I will
provide wiki guides for tuning Linux to get the most performance, as I
discover while bench testing the software.

I will develop a performance regression suite that anyone can use, and if
possible allow people to run their own tests on the hardware.

I give **no guarantees**. I may take this hardware and slack off for months at a
time. Odds are pretty good I'll make use of it most every day though.

## What else? ##

I also maintain or work on many other projects that are widely used for
internet scalability, primarily MogileFS and other Danga projects. If helpful,
I could also use the hardware to test those projects and give them many more
improvements.


---

# How to Contribute? #

Back of the napkin cost looks to be between $9,000 and $14,000 USD. More money
means more of the bonuses get filled. This is a rough estimate as people may
get steep discounts with their hardware vendor, or one may be feeling
charitable and help with the cost more. If you can get a four socket modern
intel box for cheaper than that, I'd love to know!

Otherwise, I have no idea. Send me an e-mail (dormando in rydia dawt net) with what
you're willing to do. Even if you couldn't afford the whole thing yourself, if
your company could pledge some amount, or convince me to toss up a
kickstarter/something page, that'd help.

Do **not** just send me money. I'll take either shipped hardware, or will wait
until enough pledges come in that I could have everyone send money to a
hardware vendor to have the machines built. It would be much less shady if I
have people pay a vendor or 3rd party, I think.

Thanks!