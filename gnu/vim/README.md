
# Search and replace in directory

```
:vimgrep /SomeString/gj **/*
:copen
:cfdo %s/SomeString/AString/g | update
```
