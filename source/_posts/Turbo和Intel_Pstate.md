---
title:  服务器server的频率知识整理
date:   2016-12-15 19:10:53 +0800
categories: Data Center
---
最近在做Turbo相关的控制，整理一下知识。

## 1 服务器的频率

当我们在讨论服务器的频率时，我们怎么沟通？工程师经常挂在嘴边：“这台机器P1频率是多少？P0n是多少？”

这里的P1频率，P0n频率，是什么意思呢？刚开始接触的时候会有点糊涂，不过理一下还是很容易理解的。如下图所示：
![](/images/pn.png)

对于CPU，有一个标称频率（Base Freqency)。比如某台服务器的CPU的Base Freqency是3.0GHz，也就是在普通状态下，BIOS关闭Turbo开关的话， CPU的最高频率不会超过3.0GHz。最左边的是Pn，指的是硬件上的最低频率，P1是上文有提过的Base Frequency。在这范围的频率就是P1，P2， P3...Pn，后面的数字越小，就说明频率越高。那么P1以上的频率就称为Turbo频率，P0就是Turbo的最高频率。Turbo就是我们所说的超频。什么叫超频，顾名思义，就是超出标称频率。《energy efficiency server》一书中对turbo的定义如下：
> Turbo is a feature that allows those workloads to run at higher frequencies while staying within the thermal and electrical specifications of the processor.

聊到频率，就不得不提，Intel的频率控制driver——Intel Pstate。这个driver是集成在内核（kernel）中的。摘取一段官方对pstate的解释：

> What is a P-state?
> When someone refers to a P-state, generally only the frequency is talked about. For example, on my Intel® Core™ processor, P0 is 2.3 GHz, and P1 is 980 MHz. In truth, a P-state is both a frequency and voltage operating point. Both are scaled as the P-state increases. 

Intel Pstate 是Linux kernel中调节频率的driver，对于现代的CPU，一般都是采用intel_pstate driver, 这个driver在Sandy Bridge（以及以后的CPU）开始启用。它有丰富的频率控制算法，让CPU的频率对不同的设置，不同的业务和需求，有着出不同的表现。在用户态（user space）可以采用cpupower等频率策略调整工具进行一些高级配置。有兴趣的同学可以研究一下kernel中pstate driver的代码。Pstate控制频率范围不仅依赖硬件，也依赖系统中的参数设置。关于linux中频率的查看，可以采用自带的turbostat工具。通过这个工具，你可以看到很多频率的相关信息。
![](/images/turbosetting.PNG)
![](/images/turbofreq.PNG)

CPU的频率不仅受CPU本身的逻辑控制，还跟power和thermal的限制有关。购买CPU的时候，你可能会注意到有一个指标叫TDP(thermal design point)。
> The TDP specifies the amount of power that the CPU can consume, running a commercially available worst-case SSE application over a significant period of time and therefore the amount of heat that the platform designer must be able to remove in order to avoid thermal throttling conditions. 

默认情况下，如果系统的power达到TDP，CPU的频率就会被限制以保护CPU的温度和功耗不过高。

当我们购买服务器部署业务的时候，都希望能够把服务器的性能压榨到极致，并且服务器又稳定不出错。假设服务器上部署了两种业务，一种是典型业务，一种是需求比较高的业务。通常的，长时间内服务器是跑典型业务，但偶尔来几个高需求业务，服务器就表示，哎呀哎呀，频率不够用呀。那怎么办呢？要么花钱升级服务器，要么有另一种办法，打开Turbo。在Intel最新一代的处理器上，打开Turbo可以提高10%~20%（甚至更多）的峰值性能提升。如果只是打开Turbo，然后随意跑你的业务的话，频率会根据你的业务，机会性（by opportunity)地增加到Turbo频率。因此，对Turbo有种误解是，Turbo只能在“超高”性能上维持非常短的一段时间，实则不然。除了达到一些thermal的极限条件，正确使用的话，CPU还是可以在turbo频率范围内的某个频率维持着，甚至可以让某单个core维持更高的频率。

## 2 控制频率的正确姿势

在控制频率之前，首先要认清自己的机器上正在使用的cpufreq控制器是哪个driver。
intel_pstate 的kernel sys fs在/sys/devices/system/cpu/intel_pstate, 如果有这个文件夹，说明kernel启用了intel_pstate driver。

intel_pstate文件夹下有几个参数：
```
cd /sys/devices/system/cpu/intel_pstate/
max_perf_pct  min_perf_pct  no_turbo      num_pstates   turbo_pct
```

他们的作用可以查看kernel的解释，这几个接口定义如下：
> max_perf_pct: Limits the maximum P-State that will be requested by
the driver. It states it as a percentage of the available performance. The
available (P-State) performance may be reduced by the no_turbo
setting described below.

> min_perf_pct: Limits the minimum P-State that will be requested by
the driver. It states it as a percentage of the max (non-turbo)
performance level.

> no_turbo: Limits the driver to selecting P-State below the turbo
frequency range.

> turbo_pct: Displays the percentage of the total performance that
is supported by hardware that is in the turbo range. This number
is independent of whether turbo has been disabled or not.

> num_pstates: Displays the number of P-States that are supported
by hardware. This number is independent of whether turbo has
been disabled or not.

这边比较常用到的是no_turbo这个开关，可以开关turbo，前提是BIOS已经enable了turbo功能。
在/sys/devices/system/cpu文件夹下面，有每个CPU的设置
```
cd /sys/devices/system/cpu/cpu
cpu0/    cpu12/   cpu16/   cpu2/    cpu23/   cpu27/   cpu30/   cpu34/   cpu38/   cpu41/   cpu45/   cpu49/   cpu52/   cpu56/   cpu6/    cpu63/   cpu67/   cpu70/   cpu74/   cpu78/   cpu81/   cpu85/   cpufreq/
cpu1/    cpu13/   cpu17/   cpu20/   cpu24/   cpu28/   cpu31/   cpu35/   cpu39/   cpu42/   cpu46/   cpu5/    cpu53/   cpu57/   cpu60/   cpu64/   cpu68/   cpu71/   cpu75/   cpu79/   cpu82/   cpu86/   cpuidle/
cpu10/   cpu14/   cpu18/   cpu21/   cpu25/   cpu29/   cpu32/   cpu36/   cpu4/    cpu43/   cpu47/   cpu50/   cpu54/   cpu58/   cpu61/   cpu65/   cpu69/   cpu72/   cpu76/   cpu8/    cpu83/   cpu87/
cpu11/   cpu15/   cpu19/   cpu22/   cpu26/   cpu3/    cpu33/   cpu37/   cpu40/   cpu44/   cpu48/   cpu51/   cpu55/   cpu59/   cpu62/   cpu66/   cpu7/    cpu73/   cpu77/   cpu80/   cpu84/   cpu9/
```

进到某个cpu thread里面可以看到也有cpufreq的文件夹，进入到cpufreq文件夹，可以看到如下一下参数：
```
cd /sys/devices/system/cpu/cpu0/cpufreq/
affected_cpus                cpuinfo_max_freq             cpuinfo_transition_latency   scaling_available_governors  scaling_driver               scaling_max_freq             scaling_setspeed
cpuinfo_cur_freq             cpuinfo_min_freq             related_cpus                 scaling_cur_freq             scaling_governor             scaling_min_freq
```

控制frequency建议的做法是用cpupower这个linux kernel tool来控制，不容易改错参数。
```
Usage:  cpupower [-d|--debug] [-c|--cpu cpulist ] <command> [<args>]
Supported commands are:
        frequency-info
        frequency-set
        idle-info
        idle-set
        set
        info
        monitor
        help

Not all commands can make use of the -c cpulist option.

Use 'cpupower help <command>' for getting help for above commands.
```

主要用的设置命令是用的frequency-set这个指令，具体用法：
```
NAME
       cpupower frequency-set - A small tool which allows to modify cpufreq settings.

SYNTAX
       cpupower [ -c cpu ] frequency-set [options]

DESCRIPTION
       cpupower frequency-set allows you to modify cpufreq settings without having to type e.g. "/sys/devices/system/cpu/cpu0/cpufreq/scaling_set_speed" all the time.

OPTIONS
       -d --min <FREQ>
              new minimum CPU frequency the governor may select.

       -u --max <FREQ>
              new maximum CPU frequency the governor may select.

       -g --governor <GOV>
              new cpufreq governor.

       -f --freq <FREQ>
              specific frequency to be set. Requires userspace governor to be available and loaded.

       -r --related
              modify all hardware-related CPUs at the same time

       REMARKS

       By default values are applied on all cores. How to modify single core configurations is described in the cpupower(1) manpage in the --cpu option section.

       The -f FREQ, --freq FREQ parameter cannot be combined with any other parameter.

       FREQuencies can be passed in Hz, kHz (default), MHz, GHz, or THz by postfixing the value with the wanted unit name, without any space (frequency in kHz =^ Hz * 0.001 =^ MHz * 1000 =^ GHz * 1000000).

       On Linux kernels up to 2.6.29, the -r or --related parameter is ignored.
```





