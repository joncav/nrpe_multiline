# nrpe_multiline
ENHANCING NRPE FOR LARGE OUTPUT

Original authorship credit to: Ton Voon & <Altinity.com>

The text below was taken from a blog article related to this patch which is no longer available online. Instead of allowing this to die, incase it was helpful I've made it available.

Original blog post, no longer working: <http://altinity.blogs.com/dotorg/2008/08/enhancing-nrpe.html>

# Enhancing NRPE

NRPE is great for getting plugin information from a remote host. We wanted to use it to get passive data regarding events, such as syslog entries that SEC had highlighted. This meant we needed two things: multi-line support and larger amounts of output.

Multi-line is already in NRPE 2.12 - this was added by Matthias Flacke last year. However, the limit for data is 1K.

We wanted to be able to bump that figure up to 16K. There's a common.h variable which is called MAX_PACKETBUFFER_LENGTH which is set to 1024. We found we could increase this value and then more data was returned. But there were two problems with it: 

- it broke backwards compatibility
- it increased the size of each packetk

The 2nd had an impact on the network. Instead of 1K packets being sent between client and server, we now got 16K packets sent, even if the data contained was small.

The first was worst: it meant you needed to update the client (check_nrpe) with the server (nrpe) at the same time, otherwise you'd get lots of NRPE errors in Nagios with only one change.

So we've designed a compatible way: we've added a new packet type called RESPONSE_PACKET_WITH_MORE.

The idea is that check_nrpe will see if the packet returned is of the type RESPONSE_PACKET_WITH_MORE. If so, it will read subsequent packets and append that to the existing data, until it gets a RESPONSE_PACKET. So to read 16K worth of data, check_nrpe reads 16 x 1K packets. Of course, only updated nrpe daemons will send this, so this remains fully backwards compatible with existing nrpe daemons.

We've also cleanup up some of the graceful_close calls.

Now the process to update your NRPE agents would be: 

update the central check_nrpe, then

update your agents at your leisure

And you won't get any alerts during this period!
Note: during testing, we found that the limit for returned data from some linux kernels was 4K, even though nrpe was coded with 16K as the limit. This is due to kernel limitations in using pipe() for the interprocess communication.
