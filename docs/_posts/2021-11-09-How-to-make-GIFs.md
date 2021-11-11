---
layout: post
title:  "How to make millions of GIFs with a GIF fuzzer"
author: Andreas Zeller
---

So you have written a program that processes GIFs. How do you test it? One common way to do so would be to collect a set of GIFs from the Internet (where there's no shortage of GIFs), and to test your program on these. Obviously, if your program fails to process one of these GIFs, you had better fix it. Here's some neat GIF from Wikipedia; you can use it to test your program.

![]({{ '/assets/Newtons_cradle_animation_book_2.gif' | relative_url }})

Unfortunately, the GIFs that are present on the Internet may not encompass the full range of features that GIFs (or your program) have to offer. As the [Wikipedia article on GIFs](https://en.wikipedia.org/wiki/GIF) will be happy to lay out, GIFs can have several obscure features such as tiles, layers, CLEAR codes, and lots of metadata. If your program is subject to third-party inputs (say, when someone uploads a GIF to your application), it had better handle all of these features –&nbsp;or at least not crash or hang. So, you need more GIFs. Where do you get them from?

In the past years, _fuzzers_ have become one of the most important tools to generate test inputs –&nbsp;in particular, to find vulnerabilities and other robustness issues. Popular fuzzers such as AFL _mutate_ a set of inputs again and again, guided by achieving a maximum _coverage_ of the program under test. 

Could you thus use such a fuzzer to test your program? The answer is: yes and no. Since a fuzzer _mutates_ given GIF files, what it will generate first and foremost is plenty of _invalid_ GIF files. You will thus thoroughly test the _parser_ that reads in a file, as well as error handling. So, yes. However, it is  unlikely that a mutation will actually create a new GIF feature, unless it is already contained in one of the given GIFs. So, no.

This is where _language_-based fuzzers come in. A language-based fuzzer such as `FormatFuzzer` uses a _format description_ called [binary template](https://www.sweetscape.com/010editor/templates.html) to generate millions of inputs that adhere to this very format. Using a binary template for GIFs, for instance, `FormatFuzzer` can generate millions of GIFs, all valid, and including all features that the GIF format has to offer. A simple

```sh
$ make gif-fuzzer
```

suffices, and you get a super-efficient GIF generator `gif-fuzzer`, which you can invoke as

```sh
$ ./gif-fuzzer fuzz foo.gif
```

to create a GIF file `foo.gif`.

Here's one of the GIFs produced by FormatFuzzer –&nbsp;[six-rectangles.gif]({{ '/assets/six-rectangles.gif' | relative_url }}), an animated series of six black rectangles. It may look unspectacular, but it covers plenty of GIF features, including animation. You can create a large number of these, and put your program to the test.

![]({{ '/assets/six-rectangles.gif' | relative_url }})

Interestingly, `six-rectangles.gif` renders differently in different browsers. On Safari 15.1, it renders as a big black rectangle:

![]({{ '/assets/GIF_Safari.png' | relative_url }})

On Firefox 94.0, it renders as a small black rectangle:

![]({{ '/assets/GIF_Firefox.png' | relative_url }})

And on Google Chrome 95.0, it renders as white space:

![]({{ '/assets/GIF_Chrome.png' | relative_url }})

So, with the help of `FormatFuzzer`, we already detected an inconsistency in how GIF files are processed. Which one is the correct behavior?

Once set up for a particular format, tools like `FormatFuzzer` can mbe tremendously useful. However, _someone has to write these binary templates in the first place_ -&nbsp;which means digging through file format specifications and encoding their rules such that `FormatFuzzer` can process them. We are currently exploring ways to _convert_ more existing formats, and also to [_mine_ such formats from existing programs](https://publications.cispa.saarland/3101/). However, [many common formats already _are_ encoded as binary templates](https://www.sweetscape.com/010editor/repository/templates/) (including GIFs!) and it only takes little effort to make them suitable for high-quality production of inputs.

GIF is one of the formats that is already fully supported by `FormatFuzzer`. Hence, you can happily produce as many GIFs as you like, and test your program against them. Happy fuzzing!



