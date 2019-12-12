---
title: ConfigureAwait FAQ
date: 2019-12-12
summary: Stephen Toub - One aspect of async/await that continues to draw questions is ConfigureAwait. In this post, I hope to answer many of them. I intend for this post to be both readable from start to finish as well as being a list of Frequently Asked Questions (FAQ) that can be used as future reference.
tags: [dotnet, link]
---

A post by [Stephen Toub](https://devblogs.microsoft.com/dotnet/author/stephen_toubhotmail-com/)

>.NET added async/await to the languages and libraries over seven years ago. In that time, it’s caught on like wildfire, not only across the .NET ecosystem, but also being replicated in a myriad of other languages and frameworks. It’s also seen a ton of improvements in .NET, in terms of additional language constructs that utilize asynchrony, APIs offering async support, and fundamental improvements in the infrastructure that makes async/await tick (in particular performance and diagnostic-enabling improvements in .NET Core).

>However, one aspect of async/await that continues to draw questions is ConfigureAwait. In this post, I hope to answer many of them. I intend for this post to be both readable from start to finish as well as being a list of Frequently Asked Questions (FAQ) that can be used as future reference.

https://devblogs.microsoft.com/dotnet/configureawait-faq/