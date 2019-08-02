---
layout: post
title: Improving performance with .NET Core 3.0
date: '2019-08-02T11:15:00.000+10:00'
author: Richard Banks
---
The week I gave a talk at the Sydney Alt.Net Meetup about using .NET Core 3.0, Span<T>, stackalloc, and other newer .NET features to improve performance of an application.

In it I showed a specific scenario where I used a PDF library ([PdfPig](https://uglytoad.github.io/PdfPig/)) to count the words in a PDF file, and in less than an hour improved the performance by over 30%.

<script async class="speakerdeck-embed" data-id="daf7b1bc3fd04e86841c0f43dad7f8b7" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

Much of the talk centered around actually making the performance improvements and shortly I'll create a video showing that side of the talk, and how PerfView and benchmarkDotNet can be used to help.

For now, you can look at the code at various stages of improvement by jumping between the different branches of my cloned PdfPig repo at https://github.com/rbanks54/PdfPig.

Here are the branches you might want to look at:

  * __master__: the code "as-is", simply the snapshot of the original library, with no changes.
  * __startPoint__: Converted to .NET Core 3.0 and with a "performanceTester" utility you can run to see overall performance for our test scenario. You can switch the csproj to netcoreapp2.2 to see the difference between Core 3.0 and Core 2.2
  * __numericTokens__: Added microbenchmarking and improved performance of the numeric token parser. Run the benchmarks program to see the numbers
  * __NameTokens__: microbenchmarking, with multiple approaches. Use the benchmarks to compare performance differences of various approaches. Run with "-m" to see the memory usage stats.
  * __both-tokens__: code with both the numeric and name token improvements in it. Run the performanceTester to compare with the original code.