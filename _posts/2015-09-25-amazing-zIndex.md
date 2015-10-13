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

从图中可以看到三个很奇怪的渲染层，html(414 * 17804), .layout-body(398 * 17537), .layout-body(398 * 17538)。 这三个渲染层的尺寸都非常大，相当于几乎页面上所有的内容都重复渲染了4遍，这对内存的占用想必不少，并且这三个渲染层的产生完全不符合预期。那就看看这三个渲染层是怎么产生的，然后再想办法消除。好在chrome给出了每个渲染层产生的原因：

- html(414 * 17804): Secondary layer, to contain any normal flow and positive z-index contents on top of a negative z-index layer.
- .layout-body(398 * 17537): Might overlap other composited content. Cannot be squashed because this layer has a different transform ancestor than the squashing layer.
- .layout-body(398 * 17538): Secondary layer, home for a group of squashable content.

chrome给的这三个原因没有看的很明白，但是大概的意思是说层级的问题，尤其是有一个negative z-index layer。那就赶紧查一下页面上哪些地方用了负值的z-index，还真找到了有个元素是负z-index。
![after][4]

.animation-item-2节点是.get-header的后代元素，.layout-body是.get-header的兄弟元素。.animation-item-2的各层父级元素都没有设置z-index，而.animation-item-2有3D变换属性，被单独提升为渲染层了，因此浏览器在渲染的时候就需要将整个html渲染两遍，并将.animation-item-2像汉堡一样夹在中间，这样才能保证渲染效果。
现在的问题应该是.animation-item-2的个父级元素没有z-index导致.animation-item-2的负值z-index是相对整个html文档了，那是不是给.animation-item-2的某个父级元素设置一个正值z-index就可以了，这样就只是这个父级节点渲染两遍来包裹.animation-item-2元素。于是就给.get-header加上了一个正值z-index，结果帅呆了！
![after][3]


###解决crash
从这个问题可以得出如下结论：
1. iPhone6 plus也不是神一样的存在了，也会存在ip6 plus only的问题；
2. 浏览器会自动根据一系列原则将某些元素单独提升为渲染层来提升渲染性能；
3. 如果一个容器里有后代元素拥有负的z-index和其他元素，并且该元素是独立渲染层，则该容器也会被提升为独立渲染层，而其他兄弟元素也会生成一个独立渲染层；
4. 负值z-index坑比较大，能不用最好还是不要用了；
5. chrome开发工具很好很强大



###补上浏览器生成独立渲染层原则
1. The layer has 3D or perspective transform CSS properties.
2. The layer is used by \<video\> element using accelerated video decoding.
3. The layer is used by a \<canvas\> element with a 3D context or accelerated 2D context.
4. The layer is used for a composited plugin, e.g. Flash or Silverlight.
5. The layer uses a CSS animation for its opacity or uses an animated webkit transform.
6. The layer uses accelerated CSS filters.
7. The layer has a descendant that is a compositing layer.
8. The layer has a sibling with a lower z-index which has a compositing layer (in other words the layer is rendered on top of a composited layer).

[更多关于compositing layer][1]


[1]: https://aerotwist.com/blog/on-translate3d-and-layer-creation-hacks/ "on-translate3d-and-layer-creation-hacks"
[2]: http://timcui.github.io/public/pic/ip6crash/before.png "before"
[3]: http://timcui.github.io/public/pic/ip6crash/after.png "after"
[4]: http://timcui.github.io/public/pic/ip6crash/nodes.png "nodes"