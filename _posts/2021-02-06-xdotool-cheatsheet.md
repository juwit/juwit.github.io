---
title: xdotool cheatsheet
created: '2021-02-06'
modified: '2022-04-15'
language: en
canonical_url: https://codeka.io/2021/02/06/xdotool-cheatsheet/
tags:
  - Tools
---

I played a lot these days with xdotool, to try to automate some stuff for my Elgato Streamdeck.

These are the things I try to do:

* Select a window, and send a Keyboard sequence (such as CTRL+B to mute/unmute a Teams call)
* Type emojis to the active window 😅
* More a window around, or resize it

Here are some links that I found about `xdotool`:

* [Xdotool - Window Stack](https://www.linux.org/threads/xdotool-%E2%80%93-window-stack.10687/)
* [Xdotool - Examples](https://www.linux.org/threads/xdotool-examples.10705/#post-36275)
 

Below are the commands I found useful during my research.

## get window classes

```
xprop | grep 'CLASS'
```

Click on the window you want to analyse after that.

More details about classes on this thread: [xdotool: what are "class" and "classname" for a window?](https://askubuntu.com/questions/1060170/xdotool-what-are-class-and-classname-for-a-window)

## find active window

```
xdotool getactivewindow
```

## find a window by class

```
xdotool search --onlyvisible --limit 1 --class "Firefox"
```

## focus a window

```
xdotool windowactivate 123456
```

## send a key

```
xdotool search --onlyvisible --limit 1 --class "Firefox" key ctrl+t
```

## send a key to firefox (by changing active window and come back)

```
ACTIVE_WINDOW=$(xdotool getactivewindow)
FIREFOX_WINDOW=$(xdotool search --onlyvisible --limit 1 --class "Firefox")
xdotool windowactivate $FIREFOX_WINDOW
xdotool key ctrl+s
xdotool windowactivate $ACTIVE_WINDOW
```

## change a window size

```
xdotool search --onlyvisible --limit 1 --class "Firefox" windowsize 800 600 
```