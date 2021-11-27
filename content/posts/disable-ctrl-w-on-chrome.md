---
title: "Disable Ctrl+w on Chrome"
date: 2021-10-09
categories:
  - other
tags:
  - chrome
  - cka
---

I am preparing the CKA exam. The exam requires us to complete tasks using web terminal on Chrome.

My concern is that Ctrl+w is a common shortcut in terminal (delete word) but it also is the "close tab" shortcut of chrome. (Windows)

There is a workaround on this issue. Comment under [this article](https://suraj.io/post/disable-ctrl-w/) mentioned that we can define ctrl+w in <chrome://extensions/shortcuts> to override the default behavior on ctrl+w. This will prevent the tab be closed when hit ctrl+w accidently.
