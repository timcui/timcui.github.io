---
layout: post
title: 神奇的z-index
category: work
tag: [javascript, crash, performance]
---

> iPhone6 plus客户端闪退，只有iPhone6 plus客户端闪退，不管是在手Q里还是手空里打开活动页面都会闪退，是不是感觉很匪夷所思？但是真的遇到了！

iPhone6 plus打开H5页面客户端就直接闪退了，根据之前类似经验，可能是CSS3样式影响了浏览器的渲染层，产生了一些垃圾层，当这些垃圾层尺寸比较大的时候就会占用比较多的内存，尤其是在iPhone6 plus的大屏上，而iPhone6 plus的内存则只有1GB而已，这样就容易导致客户端crash。嗯，很完美的猜测，那是不是该验证一下。

###crash的原因
![before][2]

从图中可以看到三个很奇怪的渲染层，html(414 * 17804), .layout-body(398 * 17537), .layout-body(398 * 17538)。 这三个渲染层的尺寸都非常大，相当于几乎页面上所有的内容都重复渲染了4遍，这对比内存的占用想必不少，并且这三个渲染层的产生完全不符合预期。那就看看这三个渲染层是怎么产生的，然后再想办法消除。好在chrome给出了每个渲染层产生的原因：
- html(414 * 17804)
- .layout-body(398 * 17537)
- .layout-body(398 * 17538): Secondary layer, home for a group of squashable content.

###补习一下浏览器生成compositing layer原则
1. The layer has 3D or perspective transform CSS properties.
2. The layer is used by \<video\> element using accelerated video decoding.
3. The layer is used by a \<canvas\> element with a 3D context or accelerated 2D context.
4. The layer is used for a composited plugin, e.g. Flash or Silverlight.
5. The layer uses a CSS animation for its opacity or uses an animated webkit transform.
6. The layer uses accelerated CSS filters.
7. The layer has a descendant that is a compositing layer.
8. The layer has a sibling with a lower z-index which has a compositing layer (in other words the layer is rendered on top of a composited layer).

[更多关于compositing layer][1]

###解决crash

![after][3]



[1]: https://aerotwist.com/blog/on-translate3d-and-layer-creation-hacks/ "on-translate3d-and-layer-creation-hacks"
[2]: http://timcui.github.io/public/pic/ip6crash/before.png "before"
[3]: http://timcui.github.io/public/pic/ip6crash/after.png "after"