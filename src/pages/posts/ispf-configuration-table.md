---
title: 'ISPF - The temporary data set qualifier is invalid.'
pubDate: 2024-03-09
description: 'Deubgging error with temporary data set qualifier.'
author: 'Baremetalfreak'
image:
    url: 'https://docs.astro.build/assets/rose.webp'
    alt: 'The Astro logo on a dark background with a pink glow.'
tags: ["astro", "blogging", "learning in public"]
---
# ISPF - The temporary data set qualifier is invalid.

Published on: 2024-03-09

I was just getting ready for a client conference when I ran into an issue with ISPF on my ZVDT instance.

## Error on `srchfor`
When I tried to run `srchfor` in `sys1.proclib` I got the following message

```
The temporary data set qualifier is invalid.
The log, list and temporary cntl data sets will be allocated without it.
***
```

I asked Aria, the Opera AI assistent, for help and it pointed me to a nice [article](http://www.meerkatcomputerservices.com/mfblog/wp-content/uploads/2017/08/ISPF-Using-ISVPCALL-3.14-Failure.pdf) that discussed the same issue. As much as it was interesting to learn about `TSO ISPVCALL` for tracing ISPF processing it did not help me much. I had the same issue for all users and so could not compare a good and bad trace. Also deleting the `&SYSUID..SPFLOGx.LIST` datasets did not help.

The ISPVCALL trace did however contain some clues. There was an error mentioning `ISPA001`. Looked into the ISPF Messages and Codes manual and it said:

```
This message precedes further messages for which ISPF system data received the allocation error.
```

So I looked further down the trace and found `ISPA008`. Checking the Messages and Codes manual again the _Explanation it_ said

```
The temporary data set qualifier, specified in the
configuration utility, did not adhere to the qualifier
naming convention.
```

## ISPF Configuration Table

So I fired up the the _ISPF Planning and Customizing_ manual and read through chapter 2 _ The ISPF Configuration Table_.

There I learned about the config table load module `ISPCFIG` and its user and VSAM variants `ISPCFIGU` and `ISPCFIGV` respectively.

Then
1. Check the `STEPLIB` concatenation of my TSO logon procedure but none of its libraries contained an `ISPCFIG` or `ISPCFIGU` load modules.
1. Look into the LNKLST concatenation. Found a `ISFCFIGU` load module in `XYZ.LIBRARY` PDS.
1. Start the _ISPF Configuration Utility_ via `TSO ISPCCONF`
1. On the initial panel specify a PDS and a member where to store the keyword file under the _Keyword File Data Set_ section _Dataset_ and _Member_ field.
1. Pick option 7 _Convert Configuration Table Loadmod to Keyword File
1. Specify `'XYZ.LIBRARY'` as _Input Data Set Name_ and `ISPCFIGU` as _Input Member_

This converted the load module back to a kewyord format specified on the initial panel.

Now
1. Select option 2 _Edit Keyword File Configuration Table_
1. Find `ISPF_TEMPORARY_DATA_SET_QUALIFIER` and see that it is defined as

```
ISPF_TEMPORARY_DATA_SET_QUALIFIER           = &SYSNAME(5:4)
```

Now it all made sense. It was most likely copied from another system which had 8 character `&SYSNAME` and only the last 4 (positions 5-8) were use. Well my system only had a 4 character name and that is why it was failing.

The fix was to change the `ISPF_TEMPORARY_DATA_SET_QUALIFIER` to
```
ISPF_TEMPORARY_DATA_SET_QUALIFIER           = &SYSNAME
```
1. Exit the editor via PF03
1. Let the utility verify the table.
1. On the main panel select option 4 _Build Configuration Table Load Module_

After logging out of TSO and logging back in all worked.

Note: This is a shortened version of the story. There was a number of try and fail steps in between.
