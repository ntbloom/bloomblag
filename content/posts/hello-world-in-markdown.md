---
author: ntbloom
title: "Does Hello World Count in Markdown?"
date: 2020-04-14
draft: true
---

Hello World is one of my favorite arcane traditions in the world of software
engineering. While it represents a seemingly simple task -- can you print a sentence on
the screen? -- it symbolizes the power of software to do amazing things.

Unlike its ugly cousins foo, bar, and baz, Hello World is a much kinder friend who
invites you to show off your newfound skills to whoever out there who might be
listening. Call me a dumb kid (I'm currently 36), but I still crack a huge smile every
time I run a new Hello World on something I've never seen before.

As my first ever blog post, it would be both irresponsible and way less fun if I didn't
follow in the footsteps of the millions of other programmers who came before me and do
my own hello world for this site.

Since this blog is written using [Hugo](https://gohugo.io "https://gohugo.io"), I'm
writing mostly in markdown (which hardly counts as coding, but I get to use vim --
woo!). But I do have a fondness for markdown syntax highlighting support, so now is as
good a time as any to show it off:

````plaintext
```py3
print("Hello World!")
```
````

translates to...

```py3
print("Hello World!")
```

That was fun, let's keep going...

```java
class Hello {
  public static void main(String[args]) {
    System.out.println("Hello, World!");
  }
}
```

```c
#include <stdio.h>
int main(void)
{
  printf("Hello, World!\n");
  return 0;
}
```

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Hello, World!</title>
  </head>
  <body>
    <p>Hello, World!</p>
  </body>
</html>
```

```javascript
console.log("Hello, world!");
```

```awk
BEGIN { print "Hello, World!" }
```

```react
import React from "react";
import ReactDOM from "react-dom";
class Hello extends React.Component {
  render() {
    return (
      <div>
        <p>Hello, World!</p>
      </div>
    );
  }
}
ReactDOM.render(<Hello />, document.getElementById('root'));
```

```rust
fn main() {
  println!("Hello, World!");
}
```

```postgresql
SELECT "Hello, World!";
```

```TeX
\documentclass[11pt,letterpaper]{article}
\begin{document}
Hello, world!
\end{document}
```

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:document
  xmlns:o="urn:schemas-microsoft-com:office:office"
  xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships"
  xmlns:v="urn:schemas-microsoft-com:vml"
  xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main"
  xmlns:w10="urn:schemas-microsoft-com:office:word"
  xmlns:wp="http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing"
  xmlns:wps="http://schemas.microsoft.com/office/word/2010/wordprocessingShape"
  xmlns:wpg="http://schemas.microsoft.com/office/word/2010/wordprocessingGroup"
  xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
  xmlns:wp14="http://schemas.microsoft.com/office/word/2010/wordprocessingDrawing"
  xmlns:w14="http://schemas.microsoft.com/office/word/2010/wordml"
  mc:Ignorable="w14 wp14"
  ><w:body
    ><w:p
      ><w:pPr
        ><w:pStyle w:val="Normal" /><w:bidi w:val="0" /><w:jc
          w:val="left" /><w:rPr></w:rPr></w:pPr
      ><w:r><w:rPr></w:rPr><w:t>Hello, World!</w:t></w:r></w:p
    ><w:sectPr
      ><w:type w:val="nextPage" /><w:pgSz w:w="12240" w:h="15840" /><w:pgMar
        w:left="1134"
        w:right="1134"
        w:header="0"
        w:top="1134"
        w:footer="0"
        w:bottom="1134"
        w:gutter="0" /><w:pgNumType w:fmt="decimal" /><w:formProt
        w:val="false" /><w:textDirection w:val="lrTb" /></w:sectPr></w:body
></w:document>
```

Wait, was that last one an uncompressed MS Word document? Get the hell out of here.
