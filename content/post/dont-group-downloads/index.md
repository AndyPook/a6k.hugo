---
title: "Windows, please don't group Downloads"
date: 2019-12-05T18:04:58Z
summary: Starting with Windows 1903, Explorer has started grouping Downloads by date... Super annoying.
---

Starting with Windows 1903, Explorer has started grouping Downloads by date. You can right-click and change the grouping. But it will always revert to group-by date. Super annoying.

In regedit find "Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FolderTypes\{885a186e-a440-4ada-812b-db871b942259}\TopViews\{00000000-0000-0000-0000-000000000000}"

Right-click; Permissions; Advanced; Change owner to "Administators"; Change permissions to "Full Control".

Finally, delete all the keys that look like the do sorting/grouping.

Solved!