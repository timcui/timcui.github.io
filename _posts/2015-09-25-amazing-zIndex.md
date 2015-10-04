---
layout: post
title: 神奇的z-index
category: work
tag: [javascript, crash, performance]
---

> iPhone6 plus客户端闪退，只有iPhone6 plus客户端闪退，不管是在手Q里还是手空里打开活动页面都会闪退，是不是感觉很匪夷所思？但是真的遇到了！

iPhone6 plus打开H5页面客户端就直接闪退了，根据之前类似经验，可能是CSS3样式影响了浏览器的渲染层，产生了一些垃圾层，当这些垃圾层尺寸比较大的时候就会占用比较多的内存，尤其是在iPhone6 plus的大屏上，而iPhone6 plus的内存则只有1GB而已，这样就容易导致客户端crash。嗯，很完美的猜测，那是不是该验证一下。

###crash的原因

###浏览器生成compositing layer原则
- The layer has 3D or perspective transform CSS properties.
- The layer is used by <video> element using accelerated video decoding.
- The layer is used by a <canvas> element with a 3D context or accelerated 2D context.
- The layer is used for a composited plugin, e.g. Flash or Silverlight.
- The layer uses a CSS animation for its opacity or uses an animated webkit transform.
- The layer uses accelerated CSS filters.
- The layer has a descendant that is a compositing layer.
- The layer has a sibling with a lower z-index which has a compositing layer (in other words the layer is rendered on top of a composited layer).

[更多关于compositing layer][1]

###解决crash




[1]: https://aerotwist.com/blog/on-translate3d-and-layer-creation-hacks/ "on-translate3d-and-layer-creation-hacks"