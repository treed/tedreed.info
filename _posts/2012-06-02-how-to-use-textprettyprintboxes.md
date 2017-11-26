---
layout: post
title: "How to use Text.PrettyPrint.Boxes"
category: programming
tags: [Haskell]
---
This'll be a short post with slightly different subject matter than my prior posts.

Recently I wrote a small utility in Haskell for [calculating ore values](https://github.com/treed/ore) in EVE Online. It determines what various ores are worth based on the value of the component minerals you can get out of it. At the end it prints a table with two columns. Well, it does now. Previously it just printed lines with `printf "%s %.2f\n"`, which doesn't line up very well.

Some googling directed me to [Text.PrettyPrint.Boxes](http://hackage.haskell.org/packages/archive/boxes/latest/doc/html/Text-PrettyPrint-Boxes.html). The documentation lists all of the functions and datatypes, but leaves the reader to figure out how they go together. I couldn't find any other examples via Google, so I ended up using ghci to play with it until I got it figured out. I figure that since I would have found this example useful, others will too. Here's the code:

{% highlight haskell %}
print_table :: [[String]] -> IO ()
print_table rows = printBox $ hsep 2 left (map (vcat left . map text) (transpose rows))
{% endhighlight %}

And here's an example of the output, given a list of lists each containing two strings:

    treed@eunice:~/code/ore$ runghc ore.hs
    Mercoxit     335.67
    Arkonor      276.15
    Bistot       214.30
    Hemorphite   213.88
    Jaspet       205.37
    Crokite      204.10
    Hedbergite   200.69
    Pyroxeres    180.26
    Dark_Ochre   170.34
    Veldspar     159.18
    Scordite     149.29
    Plagioclase  144.11
    Kernite      133.00
    Gneiss       94.26
    Omber        78.62
    Spodumain    77.95

So, as you can see, the function takes a list of lists of Strings, denoted as `rows`. The first thing it does is transpose it so that you have a list of columns instead. This is because Boxes won't be able to format things correctly if you construct row-wise. You need to construct columns and then combine those for it to actually align correctly.

So first I turn the rows into columns, I then map each column with a Boxes function `vcat`, which simply takes a list of boxes (here made by mapping the `text` function over the entries in the column), and concatenates them together into a new box vertically, with an alignment (`left`, in this case).

Having thus generated a list of vcatted columns, I then combine them with `hsep`, which is a convenience form of `hcat` which also takes a number representing the minimum whitespace between columns. Again I give it a `left` alignment.

The output of `hsep` goes to `printBoxes`, which does the actual output to stdout.

Now, one drawback here is that it applies the same alignment to everything. It'd be nice to be able to specify alignments, perhaps on a per-column basis. (The value column really ought to be right-aligned, for instance.) Perhaps I'll get to that soon. But for now I have a nice simple function that prints out a nicely formated table.

You could also, if you're more of a purist bent, or want to be able to test it, change the type to `[[String]] -> Box`, and do the printBox elsewhere. In which case, I'd probably rename the function to `construct_table` or some such.
