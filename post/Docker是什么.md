---
title: Docker是什么
date: 2016-12-27 12:40:10 
categories: Docker 
tags: [Docker] 
description: 
---

简单说它是一个开源的容器引擎，可以帮助开发者高效的构建应用。
<!--more--> 

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=437250607&auto=0&height=66"></iframe>

我们先看一下正常情况，我们是怎样构建一个简单应用的呢？首先它要有一个基础的平台，也就是操作系统来让它运行，比如 win，linux 等等。然后我们就可以写代码来实现它。在这个过程中，我们可能会用到数据库，框架等等。为了在代码中使用这些，我们会根据需要进行相关的配置，或者下载所需的依赖。但这个过程有时并不是想象中的那么简单，甚至有些时候仅仅是配置环境就让一些开发者望而却步。等我们所有的工作都完成后，将它部署到服务器上提供服务，我们的应用就开发完了，接着就是无休止的运维了。

为了更好的理解 Docker 是什么？我们把上面构建应用的过程比作运输货物，我们把上面提到的操作系统看作是进行运输的交通工具，这里我们就把大鲸鱼当作交通工具吧，把交付的应用程序看成是各种货物，我们要将各种各样形状、尺寸不同的货物放到大鲸鱼上，我们需要为每件货物考虑怎么安放（就是应用程序配套的环境），还得考虑货物和货物是否能叠起来（应用程序依赖的环境是否会冲突）。这可不是一份简单的差事，有时候因为安排不当甚至会导致这一次运输的失败。但后来出现了集装箱，我们把每件货物都放到集装箱里，这样我们的就可以用同样地方式安放、堆叠集装了，省事省力。

上面提到的集装箱就是 Docker 中的“容器”，而 Docker 就是管理这些集装箱的一整套机制。集装箱好像只是做了一层封装，没有什么很神奇的地方。但我们继续想象下场景，集装箱出现之后，世界上绝大多数的货物运输都可以放到这个神奇的箱子里，然后在公路、铁路、海洋等所有运输场景下，这个箱子都不用变化形态直接可以承运，而且中间的中转工作，都可以通过大型机械搞定，效率大大提升，从此生产力飙升。因为集装箱规范了运输的标准，于是相应的船舶、卡车、列车以及自动化中转设备才能按照规格，被制造出来，然后使联运以及自动化成为可能，才可以极大的提高效率，提升自动化水平。集装箱本身是一个产品，而这个产品无非就是标准化的具体体现，现实世界中的事实显而易见，就是这么简单。

按照这个思路，Docker 其实跟集装箱一样，或者说它想跟集装箱一样，成为穿着马甲的“标准化”。这样开发工程师就可以把它们开发出来的 bug 们放到“集装箱”里，然后运维人员就可以使用标准化的操作工具去运维这些可爱的 bug 们。于是实现了“海陆联运”，就好像运维工程师根本不需要了解其运维的软件架构而开发工程师也并不需要了解其软件运行的操作系统一样...... 总的来说，Docker 的目的是实现自动化运维，自动化运维的大前提是标准化，而 Docker 就是实现标准化的工具。

然后我们具体看看它能给开发和运维带来哪些福利。

它可以让我们更快速的交付和部署应用。使用 Docker，开发人员可以使用镜像来快速构建一套标准的开发环境；开发完成之后，测试和运维人员可以直接使用相同环境来部署代码。Docker 可以快速创建和删除容器，实现快速迭代，大量节约开发、测试、部署的时间。并且，各个步骤都有明确的配置和操作，整个过程全程可见，使团队更容易理解应用的创建和工作过程。

它可以实现更高效的资源利用。Docker 容器的运行不需要额外的虚拟化管理程序支持，它是内核级的虚拟化，可以实现更高的性能，同时对资源的额外需求很低。

它能帮我们更轻松的迁移和扩展。Docker容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑、服务器等。 这种兼容性让用户可以在不同平台之间轻松地迁移应用。同时 Docker 创造性的使用了类似 git 管理代码的方式对镜像进行管理，也方便我们进行获取和管理 Docker 镜像。

它可以帮助我们更简单进行更新管理。使用 Dockerfile，只需要小小的配置修改，就可以替代以往大量的更新工作。并且所有修改都以增量的方式进行分发和更新，从而实现自动化并且高效的容器管理。Dockerfile 就是你的文档，并且用来产生镜像。要改变 Docker 镜像中的环境，先改 Dockerfile，用它产生镜像就行了，保证文档和环境一致。
