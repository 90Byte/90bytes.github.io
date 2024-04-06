---
title: "Post: UDP Packet Tampering"
last_modified_at: 2021-06-16T09:54:00+0900
categories:
  - Blog
tags:
  - Network
  - UDP
---
A friend recently inquired about the reasons why UDP packets could be tampered with.

It's widely known that UDP is more unreliable than TCP. It lacks error correction processes, as well as flow control and acknowledgments, thus not guaranteeing the order or the transmission of packets.

However, the conversation revealed that the question was specifically about why UDP packets could be tampered with, not just lost or damaged in transit. The existence of Frame Check Sequence (FCS) at Layer 2 made us wonder why packets could still be received in a tampered state at higher layers. Ideally, if all FCS mechanisms worked perfectly, packets might be lost but shouldn't be tampered with. I hadn't really thought about such a question before, which is somewhat embarrassing as someone with a background in networking.

Driven by curiosity, I did some research. I revisited [this question](https://networkengineering.stackexchange.com/questions/29384/why-are-there-error-detection-methods-in-almost-every-layer-of-a-network) and other network-related materials, which reminded me of several points.

There are multiple reasons why such incidents could happen:

1. As data traverses through numerous nodes, there's a non-zero chance of a packet being tampered with in a way that accidentally passes the FCS as a false positive. The FCS, using CRC, is merely a tool for detecting physical layer errors and deciding whether to drop a packet. Theoretically, a false positive occurs with a probability of 1/2^n (n=number of bits in FCS). Though practically, the probability is lower due to the address field, quantum probabilities also occur in reality, making such tampering not entirely impossible.

2. Not all Layer 2 nodes have the FCS feature enabled. Many switches and routers may ignore the existing FCS and compute a new one, potentially aggregating incorrect frames into an erroneous packet. Since each layer operates independently without interfering with other layers, higher layers can receive incorrect data as long as the header appears intact.

While this may not cover all possible reasons, the factors mentioned above seem to primarily contribute to the possibility of packet tampering.