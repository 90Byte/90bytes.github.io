---
title: "Post: RFC Repository"
last_modified_at: 2021-06-11T09:44:00+0900
categories:
  - Blog
tags:
  - RFC
  - IETF
  - Internet
  - Standards
---
# RFCs

RFC stands for Ready-For-Comments Documents, which are memoranda documenting technical specifications and policies applicable to the Internet. Although they might seem less authoritative due to occasionally hosting April Fool's jokes, organizations such as the IETF often directly adopt some RFCs as standards. Well-known technologies like JWT are also listed as proposed standards within RFCs.

# Main Content

When I was studying PKI, I heavily referred to RFCs but initially relied merely on Google due to my unfamiliarity. It was a tough job finding documents that were updated or obsolete. Later, I discovered a repository introduced by the IETF that was quite accessible. ~~Indeed, ignorance makes for harder work~~. Since then, it has been a go-to resource for me, and I highly recommend it for anyone needing to read RFC documents.
[Let's dive in.](https://www.rfc-editor.org/search/rfc_search.php)

![](/assets/images/2021/06/image-2.png)

Upon entering the site, immediately search using the desired keywords. I looked up CMP. It's advisable to search by the full name, not acronyms, to avoid excessive results. Due to the specificity of titles – likely because many who set standards use them – it's important to read them thoroughly to ensure they match what you're after.
In my case, I wished to explore the specifications for CMP itself, so I navigated to RFC 4210. At this moment, disregard whether it's the latest document; simply confirm if it meets your needs, and if it holds the status of Proposed Standard[^1].

[^1]: According to [RFC7127](https://datatracker.ietf.org/doc/html/rfc7127), an RFC must pass through three phases to become an Internet standard: Proposed Standard, Draft Standard, and Internet Standard. Hence, view a Proposed Standard as the early stage of a standard. Practically, even Proposed Standards can often be considered nearly de-facto standards in terms of maturity.

![](/assets/images/2021/06/image-3.png)

As evident, there's a straightforward exposition along with hyperlinks.

'Obsolete' signifies an entirely re-written RFC and 'update' denotes an RFC that brings in a few modifications. While obsoleted RFCs encompass the whole spec, updated RFCs might just insert the alterations.

Thus, 'obsoletes' points to the RFCs this RFC replaces. If it's marked 'updates', those are the RFCs this one has updated.
'Updated by' refers to RFCs that have updated this RFC's content. If noted as 'obsoleted by', it's the link to the new RFC that supersedes this one.
Simply select your preferred format under file formats to view the content.

Does this mean one needs to trace back through updated RFCs to gather everything wanted on the spec? Not necessarily. From what I've observed, most documents thoroughly introduce changes in the Introduction chapter.

>    This specification obsoletes RFC 2510.  This specification differs
   from RFC 2510 in the following areas:
>
>>      The PKI management message profile section is split into two
>>      appendices: the required profile and the optional profile.  Some
>>      of the formerly mandatory functionality is moved to the optional
>>      profile.
>>
>>      The message confirmation mechanism has changed substantially.
>>
>>      A new polling mechanism is introduced, deprecating the old polling
>>      method at the CMP transport level.
>>
>>      The CMP transport protocol issues are handled in a separate
>>      document [CMPtrans], thus the Transports section is removed.
>>
>>      A new implicit confirmation method is introduced to reduce the
>>      number of protocol messages exchanged in a transaction.
>>
>>      The new specification contains some less prominent protocol
>>      enhancements and improved explanatory text on several issues.

This way. Just thoroughly read and mine the necessary information.