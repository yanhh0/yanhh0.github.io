---
title: My CSS Cheatsheet
---

## user-select

The user-select CSS property controls whether the user can select
text. This doesn't have any effect on content loaded as chrome, except
in textboxes.

```css
/* Keyword values */
user-select: none;
user-select: auto;
user-select: text;
user-select: contain;
user-select: all;

/* Global values */
user-select: inherit;
user-select: initial;
user-select: unset;

/* Mozilla-specific values */
-moz-user-select: none;
-moz-user-select: text;
-moz-user-select: all;

/* WebKit-specific values */
-webkit-user-select: none;
-webkit-user-select: text;
-webkit-user-select: all; /* Doesn't work in Safari; use only
                             "none" or "text", or else it will
                             allow typing in the <html> container */

/* Microsoft-specific values */
-ms-user-select: none;
-ms-user-select: text;
-ms-user-select: element;
```
