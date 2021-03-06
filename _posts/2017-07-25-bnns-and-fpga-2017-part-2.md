---
layout: post
title: "Binarized neural networks and FPGA 2017 papers (part 2)"
date: 2017-7-25
comments: true
tags: [FPGA, deep learning, paper review, BNN]
---

In this post I will explore [binarized neural networks (BNNs) {% include
fa.html name='youtube' %}][bnn] and recent publications related to BNNs in
[FPGA 2017][fpga2017].  BNNs are popularized by Courbariaux *et al.* in this
[paper {% include fa.html name='file-pdf-o' %}][courbariaux].  The essence
of BNNs is that we constrain the majority of weights and activation values
in a deep neural network with binary values, either +1 or -1.  This allows
us to replace expensive arithmetic floating-point operations (multiplication
and addition) with highly efficient binary variants: [XNOR {% include fa.html
name='wikipedia-w' %}][xnor] (negated exclusive OR) and [popcount {% include
fa.html name='wikipedia-w' %}][popcount] (counting the number of non-zero
values).  Such architecture is highly rewarding when implemented onto FPGAs
than GPUs and CPUs: each of such operations can be easily implemented using
tiny amount of LUTs in FPGAs, and the significantly reduced size of weight
parameters further gets rid of the off-chip RAMs and the associated [von
Neumann bottleneck {% include fa.html name='wikipedia-w' %}][neumann].  This
advantage makes FPGAs shine in terms of power, speed and potentially cost,
differentiating themselves from other computing machines such as CPUs and GPUs
for implementing BNNs.

An immediate difficulty that follows binary weight and activation constraints
is that the gradient of the loss function cannot be back-propagated in the
network.  As the activation function \\( f(x) \\) binarizes values from
pre-activation, which casts continuous inputs into quantized results, this
function is not continuous:

$$
    f(x) = Sign(x) = \left\{
        \begin{aligned}
            & +1 & & \text{if } x \geq 0, \\
            & -1 & & \text{otherwise}.
        \end{aligned}
    \right.
$$

We can see that this function doesn't have a gradient value when $$ x = 0 $$,
and remains constant $$ 0 $$ at other positions. Without meaningful gradients,
it becomes impossible to train this network, as stochastic gradient descent
does not have a *gradient* to descend.  For this problem [Bengio *et al.* {%
include fa.html name='file-pdf-o' %}][bengio13] suggest that we could instead
propagate the gradient "straight-through".  In other words, it means that
we ignore the gradient contribution of the non-linear activation function,
effectively treating the activation function as an identity function with a
constant gradient of $$ 1 $$, this way the back-propagated gradient remains
continuous.  BNNs from Courbariaux *et al.* employ saturation in addition,
such that if the parameter $$ r $$ to be adjusted by its gradient $$ g_r $$
is outside the range of $$ [-1, 1] $$, we cancel its gradient by setting the
gradient to $$ 0 $$, to ensure $$ r $$ saturates at $$ \pm 1 $$.

[XNOR-Net {% include fa.html name='file-pdf-o' %}][xnor-net] from [Rastegari
*et al.*][allenai] proposes an almost identical network structure, with slight
differences in the ways they binarize weights and activations.  Instead of
using the straight-through gradient estimator we have discuss above, they
directly quantize weights and activations, and introduce scaling factors of
binary computations to closely approximate real-valued computations.

<!--more-->

# FPGA 2017 BNN papers

The paper ["Accelerating Binarized Convolutional Neural Networks with
Software-Programmable FPGAs" {% include fa.html name='file-pdf-o' %}][zhao17]
that I am going to review is from Ritchie Zhao *et al.*, which proposes a
new accelerator architecture for BNNs on a [Zynq 7Z020][zynq] FPGA.  The
illustration of the architecture, from their paper, is shown in the figure
[below](#zhao17-arch).

![Architectural diagrams of the BNN accelerator][zhao17-arch]{:
.center-image #zhao17-arch }

The diagram on the left shows the high-level control- and data-flow hierarchy,
the one on the right shows the internal components of the `Bin-Conv` unit.  The
authors recognized that because BNNs use 1-bit inputs in convolution, line
buffers in convolution must be designed with consideration of a wide-range of
image widths.  The authors therefore proposes the line buffers to have variable
widths, and adapt to different configurations for various input widths, as
shown in the figure [below](#zhao17-linebuf) they produced in the publication.
This allows them to maintain high throughput for distinct sizing of convolution
layers.

![Example usage of the variable-width line buffer][zhao17-linebuf]{:
.center-image #zhao17-linebuf }

So what is the main takeaway from this?  From the comparison results
(figure [below](#zhao17-results)) against other RTL- and OpenCL-based FPGA
implementations, I think personally it signals the viability of [high-level
synthesis (HLS)][hls] in the context of linear algebra applications.  Although
HLS does not give you the same extent of design flexibility when compared to
RTL, the low design costs associated with HLS could still vastly outweigh the
disadvantage of a slightly lower performance.

![Comparisons][zhao17-results]{: .center-image #zhao17-results}

The next paper ["FINN: A Framework for Fast, Scalable Binarized Neural Network
Inference" {% include fa.html name='file-pdf-o' %}][finn] from Umuroglu *et
al.* presents a similar approach to implement binarized architecture on FPGAs.

Their FPGA implementation remaps convolution into matrix-multiplication, and
further interleaves multiple image channels to be processed by interleaving
filters to increase parallelism.  This is illustrated [below](#finn-conv).

![Convolution using interleaved channels][finn-conv]{:
.center-image #finn-conv }

With the recent trend of realizing deep neural networks using high-level
synthesis, the authors follow suit by using [Vivado HLS][vhls] in their work
and produce results that rival prior work in RTL implementations.


# My thoughts

The current accuracy of deep neural networks is largely enabled by a
vast number of parameters introducing redundancy.  Optimization of these
networks by reducing this redundancy, also known as network compression,
is the key to future breakthroughs in running them efficiently in all
computing devices.  Such techniques include pruning and quantizing network
parameters via [deep compression {% include fa.html name='file-pdf-o'
%}][deep-compression], [training neural networks at a low precision {% include
fa.html name='file-pdf-o' %}][low-prec] and others.

BNNs, owing to their minimal weight representation ($$ \pm 1 $$), also
achieve network compression via low precision, although much more extreme
than other approaches.  I believe for this reason, their applicability has
thus been limited to networks that are already redundant in the number of
parameters, so that the *capacity* (ability to learn) of the algorithm greatly
exceeds the *complexity* (difficulty to learn) of the problem at hand, such
as [AlexNet {% include fa.html name='file-pdf-o' %}][alexnet] on [ImageNet
classification][ilsvrc], [MNIST][mnist], [CIFAR-10][cifar10] and [SVHN][svhn].

[DoReFa-Net {% include fa.html name='github' %}][dorefa] [{% include fa.html
name='file-pdf-o' %}][dorefa-pdf] seeks to generalize the extreme bitwidth
compression in BNNs by extending the formulation to multi-bit weights and
activations.  The authors show that for networks with sufficient capacity, we
can get away with binary weights and 4-bit activation, their minimal ResNet-18
with such configurations manages a 59.2% top-1 accuracy.

The problem here, I believe, lies within the tug war between performance and
network capacity.  A sweet spot necessitates achieving the optimal performance
given a required capacity constraint.  This problem is unfortunately very
difficult to answer, as we don't have a way to understand network capacity
in a well-defined form for now.  If we're in a competition to optimize DNNs
with no experiences to help us, luck would eventually becomes irrelevant in
the long run ([regression to the mean {% include fa.html name='wikipedia-w'
%}][regress-mean]), only the most productive ones at trial-and-error would
prevail.  So from now I should procrastinate less and work more.


[bnn]: https://www.youtube.com/watch?v=Q17HwA5oY4w
[fpga2017]: http://isfpga.org/fpga2017/index.html
[xnor]: https://en.wikipedia.org/wiki/XNOR_gate
[popcount]: https://en.wikipedia.org/wiki/Hamming_weight
[neumann]: https://en.wikipedia.org/wiki/Von_Neumann_architecture#Von_Neumann_bottleneck
[bengio13]: https://arxiv.org/pdf/1308.3432.pdf
[xnor-net]: https://arxiv.org/abs/1603.05279
[allenai]: http://allenai.org/plato/xnornet/
[zhao17]: http://www.csl.cornell.edu/~zhiruz/pdfs/bnn-fpga2017.pdf
[courbariaux]: https://arxiv.org/pdf/1602.02830.pdf
[zynq]: https://www.xilinx.com/products/silicon-devices/soc/zynq-7000.html
[zhao17-arch]: /assets/images/zhao17-arch.png
[zhao17-linebuf]: /assets/images/zhao17-linebuf.png
[zhao17-results]: /assets/images/zhao17-results.png
[finn]: https://arxiv.org/abs/1612.07119
[finn-conv]: /assets/images/finn-conv.png
[vhls]: https://www.xilinx.com/products/design-tools/vivado/integration/esl-design.html
[deep-compression]: https://arxiv.org/abs/1510.00149
[low-prec]: https://arxiv.org/abs/1412.7024
[alexnet]: https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks
[ilsvrc]: http://www.image-net.org/challenges/LSVRC/
[mnist]: http://yann.lecun.com/exdb/mnist/
[cifar10]: https://www.cs.toronto.edu/~kriz/cifar.html
[svhn]: http://ufldl.stanford.edu/housenumbers/
[dorefa]: https://github.com/ppwwyyxx/tensorpack/tree/master/examples/DoReFa-Net
[dorefa-pdf]: https://arxiv.org/pdf/1606.06160
[regress-mean]: https://en.wikipedia.org/wiki/Regression_toward_the_mean
