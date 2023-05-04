---
title: "Wireshark: Customization, HTTPS Traffic Decryption and Common Display Filters"
date: 2023-04-19
tags:
tags:
- "wireshark"
- "tls"
- "display-filter"
---

1. Changing [Column Display](https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/)

2. Decrypting [HTTPS Traffic](https://unit42.paloaltonetworks.com/wireshark-tutorial-decrypting-https-traffic/):
    - Close all Chrome instances
    - Start Chrome with custom param to log certificates to ssh-key.log
    - Reference ssh-key.log in Preferences > Protocols > TLS > (Pre)-Master-Secret log

3. Common Display Filter [Examples](https://wiki.wireshark.org/DisplayFilters#examples)
