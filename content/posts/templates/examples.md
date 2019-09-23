---
title: Markdown Examples
description: Include also Hugo shortcode examples
tags: [hugo]
lang: en
publishdate: 2099-12-31
---

## Code Figure With Shortcode

{{< figure "Some description with a [link to GitHub](https://github.com) in Markdown style and \"escaped quote signs\"" >}}
```js
var dummy = 123;
```
{{< /figure >}}

{{< figure "caption" >}}
```html
var dummy = 123;
```
{{< /figure >}}


## Image Figure

{{< figure "caption" >}}
![altText](linkToImage)
{{< /figure >}}


## Code Figure The Normal Way In Hugo

<figure>
{{< highlight "js" >}}
var dummy = 123;
{{< /highlight >}}
<figcaption>Some description with a <a href="https://github.com">link to GitHub</a> in Markdown style and "escaped quote signs"</figcaption>
</figure>