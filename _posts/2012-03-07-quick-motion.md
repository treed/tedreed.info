---
layout: post
title: "Quick Motion"
category: setup
tags: [vim]
---
For my first contentful post, I thought I'd do a quick runthrough of how I remapped keys in vim to enable moving around more easily.

If you've used vim for any length of time, you're probably aware of `<Leader>`. In short, it's a keymap variable which can be set with `let mapleader=` in your vimrc. By default, it's set to backslash. Map commands can then use `<Leader>` to define mappings using that variable. If `mapleader` is later redefined, it does not change the mappings already defined.

When I first began customizing my vimrc, I thought that this would be a good key to use for quick keystrokes, but found that plugins would often tromp all over the Leader mappings. I found myself having notable lag after entering what I'd intended as a quick command, which was vim waiting to see if I'd complete an ambiguous Leader map.

Of course, you can tune the timeout. You can also rearrange things such that plugins have one Leader and your own mappings have a different one. It's this last which made me realize that I didn't really want to use Leader. I just wanted a key I could press before others which would be otherwise unused and give me access to the things I do all the time.

So instead, I just used `<Space>`. I'd previously set mapleader to space, but let it revert back to backslash when I decided that I wanted to separate Leader from my own quick commands.

So, here's a basic set of the commands I ended up mapping.

{% highlight vim %}
" Open a new split
nnoremap <silent> <Space>s :sp<CR>
" Close
nnoremap <silent> <Space>q :q<CR>
" Move around between splits
nnoremap <Space>h <C-W>h
nnoremap <Space>l <C-W>l
nnoremap <Space>j <C-W>j
nnoremap <Space>k <C-W>k
" Cycle through buffers in the current split
nnoremap <Space>n :bn<CR>
nnoremap <Space>p :bp<CR>
{% endhighlight %}

Now, for some of these (all of the window motion ones), it seems like my remapping shouldn't be that much faster. It does, however, prevent you from having to reach over to press control. You can leave your fingers all in the home position, and just tap two keys in quick succession. Once muscle memory sets in, it's basically instant, because your fingers are already on those keys. If you spend all day opening and closing splits and moving around between them, optimizing this tiny thing does make a difference.

It's worth noting that if you want to use the buffer cycling, you'll also neet `set hidden` so that vim will allow you to cycle away from buffers which may not have been saved. (By default it tries to close buffers if they aren't visible anymore.) 

Also, the `<silent>` you see with the split and q commands? I put that there because otherwise, 'sp' and 'q' respectively end up sitting at the bottom of your screen until something else takes them away, and that bothers me.

I have several others of these quick commands, but they mostly rely on plugins, which I'll get to in another post.
