---
title: Stop VS2012 shouting!
date: 2012-12-12T13:24:00Z
summary: STOP VS USING ALL CAPS!
---

 1. Start PowerShell
 1. copy/paste the following
Set-ItemProperty -Path HKCU:\Software\Microsoft\VisualStudio\11.0\General -Name SuppressUppercaseConversion -Type DWord -Value 1

Next time you start VS the menus will be in mixed case.

See http://blogs.msdn.com/b/zainnab/archive/2012/06/14/turn-off-the-uppercase-menu-in-visual-studio-2012.aspx for more detail.