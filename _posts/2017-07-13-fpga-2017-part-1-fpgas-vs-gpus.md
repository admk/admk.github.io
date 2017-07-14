---
layout: post
title: "FPGA 2017 (part 1): FPGAs versus GPUs in deep learning"
date: 2017-7-13
comments: true
tags: [FPGA, deep learning, paper review]
---

Since the rapid surge in popularity, deep learning has been successfully
applied to many areas, such as [visual recognition of object categories in
images][ilsvrc], [predicting the toxicity of chemicals][toxicity], [mitosis
detection in cancer cells][mitosis], [automated student essay scoring][aes],
colorizing [artworks][color-manga] and [photos][color-photo], and [sketch
drawing simplification][sketch], and has even surpassed human
expert performance on some of the tasks.

Because of the computational intensive nature of deep learning algorithms,
traditionally GPUs are used to accelerate both the training and the test
phases.  FPGAs, with their high flexibility in terms of implementing
algorithms, could potentially achieve even higher performance and energy
efficiency than GPUs.  It came as no surprise that [the 25th ACM/SIGDA
International Symposium on Field-Programmable Gate Arrays][fpga2017] had
[two sessions][fpga2017-program] focusing on deep learning on FPGAs.  In the
following posts I will select a few interesting publications from FPGA 2017 and
review them here.

# Can FPGAs Beat GPUs in Accelerating Next-Generation Deep Neural Networks?

The [first paper {% include fa.html name='fa-file-pdf-o' %}][nurvitadhi17] with
the above title that I am going to review is written by Nurvitadhi *et al.*
from Intel.  The authors pitch their latest [Arria 10][arria10] and [Stratix
10][stratix10] devices against [Titan Xp][titanxp] in the deep learning arena,
and show that Stratix 10 has between 10% and 5.4x better performance than Titan
Xp in terms of common [GEneral Matrix to matrix Multiplication (GEMM)][gemm]
operations, which is [at the heart][warden15] of *GPU-based* deep learning
algorithms.

For the case studies, the authors start by evaluating the two FPGAs and Titan
Xp for dense and sparse GEMM with single-precision.  Titan Xp with a 11 TFlops
theoretical peak performance, has certainly greater throughput than Stratix 10,
which is rated 10 TFlops theoretical peak performance.  Their positions however
reversed as we consider the sparse GEMM, as Titan Xp performed worse than the
dense variant, and Stratix 10, with a greater performance at approximately
16.5 tera operations per second, wins the GEMM competition for deep neural
networks.  Additionally, with the advantage of hardware specialization, Stratix
10 triumphs in terms of GEMMs with reduced precision data-types such as 6-bit
integers and binary (1-bit) and ternary (2-bit) representations.  The [figure
below](#nurvitadhi17) shows the RTL templates Nurvitadhi *et al.* used to
generate FPGA GEMM implementations.

![The RTL templates Nurvitadhi *et al.* used to generate FPGA GEMM
implementations.](/assets/images/nurvitadhi17.png){:
.center-image #nurvitadhi17 }

I believe this paper did a good job in justifying FPGAs for deep learning
applications.  However, there is an important limiting factor to look at for
end users, and could be the ultimate reason why many would still consider
GPUs in favor.  This factor, which the authors from Intel didn't mention,
is the cost.  Although Stratix 10 could be 50% faster in sparse GEMM, its
[development kit][stratix10-devkit] is currently priced at $8,000, which is
6.67x that of a [Titan Xp][titanxp-price].  This places Stratix 10 at 2.06
GOP/s/$, whereas Titan Xp is at 9.07 GOP/s/$!  Although the cost-effectiveness
result favors Stratix 10 in binary neural networks (BNNs), *e.g.*, [XNOR-Net {%
include fa.html name='fa-file-pdf-o' %}][xnor-net], BNNs still cannot match the
accuracy of a standard deep neural network in tasks such as [ILSVRC][ilsvrc].
Moreover, the high development cost of FPGA applications further exacerbates
the situation.

Personally I don't think that the low cost-effectiveness of current generation
FPGAs would ultimately deter users.  On the contrary, because we are still
early adopters of deep learning in FPGAs, it foretells the great potential of
FPGAs in improving its future viability in deep learning, and FPGAs may even
takes the place of GPUs as the main workforce in both inference and training.
I have great hopes for my upcoming research direction, and expect fierce
competition from other fellow FPGA researchers.


[ilsvrc]: http://www.image-net.org/challenges/LSVRC/
[toxicity]: http://www.nature.com/nbt/journal/v33/n9/full/nbt.3299.html
[mitosis]: http://people.idsia.ch/~ciresan/data/isbi2014.pdf
[aes]: https://www.kaggle.com/c/asap-aes
[color-manga]: http://kvfrans.com/coloring-and-shading-line-art-automatically-through-conditional-gans/
[color-photo]: http://richzhang.github.io/colorization/
[sketch]: http://hi.cs.waseda.ac.jp/~esimo/en/research/sketch/
[fpga2017]: http://isfpga.org/
[fpga2017-program]: http://isfpga.org/program.html
[arria10]: https://www.altera.com/products/fpga/arria-series/arria-10/overview.html
[stratix10]: https://www.altera.com/products/fpga/stratix-series/stratix-10/overview.html
[titanxp]: https://www.nvidia.com/en-us/geforce/products/10series/titan-xp/
[nurvitadhi17]: http://jaewoong.org/pubs/fpga17-next-generation-dnns.pdf
[gemm]: https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms#Level_3
[warden15]: https://petewarden.com/2015/04/20/why-gemm-is-at-the-heart-of-deep-learning/
[stratix10-devkit]: https://www.altera.com/products/boards_and_kits/dev-kits/altera/kit-s10-fpga.html
[titanxp-price]: https://www.nvidia.com/en-us/geforce/products/10series/geforce-store/
[xnor-net]: https://arxiv.org/pdf/1603.05279.pdf
