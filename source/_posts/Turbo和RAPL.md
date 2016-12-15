---
title:  Turbo 和 Intel Pstate
date:   2016-07-13 19:10:53 +0800
categories: Data Center
---
## 关于Turbo

最近在做Turbo相关的控制，整理一下知识。

Turbo就是我们所说的超频。什么叫超频，顾名思义，就是超出某个频率。对于CPU，有一个标称频率（Base Freqency)。比如某台服务器的CPU的Base Freqency是3.0GHz，也就是在普通状态下，BIOS关闭Turbo开关的话， CPU的最高频率不会超过3.0GHz。聊到频率，就不得不提，Intel的频率控制driver——Intel Pstate。这个driver是集成在内核（kernel）中的。摘取一段官方对pstate的解释：
What is a P-state?

When someone refers to a P-state, generally only the frequency is talked about. For example, on my Intel® Core™ processor, P0 is 2.3 GHz, and P1 is 980 MHz. In truth, a P-state is both a frequency and voltage operating point. Both are scaled as the P-state increases. 

Intel Pstate中有丰富的频率控制算法，让CPU的频率对不同的设置，不同的业务和需求，有着出不同的表现。有兴趣的同学可以研究一下kernel中pstate driver的代码。
Pstate控制频率范围不仅依赖硬件，也依赖系统中的参数设置。我们平常用来沟通时说这是P1频率，这是Pn频率，是什么意思呢？刚开始接触的时候会有点糊涂，不过理一下还是很容易理解的。如下图所示：
![](/images/pn.png)

最左边的是Pn，指的是硬件上的最低频率，P1是上文有提过的Base Frequency。在这范围的频率就是P1，P2， P3...Pn，后面的数字越小，就说明频率越高。那么P1以上的频率就称为Turbo频率，P0就是Turbo的最高频率。关于linux中频率的查看，可以采用自带的turbostat工具。通过这个工具，你可以看到很多频率的相关信息。
![](/images/turbosetting.png)
![](/images/turbofreq.png)

假设服务器上部署了两种业务，一种是典型业务，一种是需求比较高的业务。通常的，长时间内服务器是跑典型业务，但偶尔来几个高需求业务，服务器就表示，哎呀哎呀，频率不够用呀。那怎么办呢？花钱买更好的服务器咯~

还有一种办法，打开Turbo。在Intel最新一代的处理器上，打开Turbo可以提高10%~20%（甚至更多）的峰值性能提升。如果只是打开Turbo，然后随意跑你的业务的话，频率会根据你的业务，机会性（by opptunity)地增加到Turbo频率。因此，对Turbo有种误解是，Turbo只能在“超高”性能上维持非常短的一段时间，实则不然。除了达到一些thermal的极限条件，正确使用的话，CPU还是可以在turbo频率范围内的某个频率维持着。如果对这块有需求，建议寻找Intel的专业支持。



