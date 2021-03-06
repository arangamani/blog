---
layout: post
category : tech
tagline: "A Vim plugin"
tags : [git, diff, vim]
---
{% include JB/setup %}

<link href="/blog/assets/css/syntax.css" rel="stylesheet" type="text/css"/>

Insert the following block of code in `.vim/plugin/gitdiff.vim`. When you want
to see the diff of the edited file and the one on the server, run the command
`:GITDiff` inside from vim.

{% highlight vim %}
if exists("loaded_gitdiff") || &compatible
    finish
endif
let loaded_gitdiff = 1
command! -nargs=? GITDiff :call s:GitDiff(<f-args>)

function! s:GitDiff(...)
    if a:0 == 1
        let rev = a:1
    else
        let rev = 'HEAD'
    endif
    let ftype = &filetype
    let prefix = system("git rev-parse --show-prefix")
    let gitfile = substitute(prefix,'\n$','','') . expand("%")
    " Check out the revision to a temp file
    let tmpfile = tempname()
    let cmd = "git show  " . rev . ":" . gitfile . " > " . tmpfile
    let cmd_output = system(cmd)
    if v:shell_error && cmd_output != ""
        echohl WarningMsg | echon cmd_output
        return
    endif
    " Begin diff
    exe "vert diffsplit" . tmpfile
    exe "set filetype=" . ftype
    set foldmethod=diff
    wincmd l
endfunction
{% endhighlight %}
