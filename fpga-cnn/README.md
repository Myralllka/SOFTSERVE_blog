# FPGA-based Convolutional Neural Network acceleration

## Introduction

We leave in the century of fast changes when the flow of information is much bigger than we can process and analyze. That is the reason why real-time systems become more and more popular. Moor's law [[Ref](https://en.wikipedia.org/wiki/Moore%27s_law)] is not working anymore from 2019: it is physically impossible to make smaller transistors because of physics, on such small sizes the Relative Theory stops working, that is time for Quantum theory and another kind of computers. <br>
That is the reason why concurrent approaches become so popular - it is easy to make it horizontal-scalable. Multicore Processors, GPU's are very popular now, but they are not suitable for some approaches. That is why we are working with FPGA's. Maybe it can be a better platform for CNN?

### What is FPGA?
**FPGA** - Field Programmable Gate Array, one more integral circuit, projected to be reconfigured after manufacturing. Always is used for ASIC's (Application Specific Integrated Circuit) prototyping, but also it is a powerful platform for making own circuits [[Ref](https://numato.com/blog/differences-between-fpga-and-asics/#what-is-asic)]. The main pros of FPGA's are creating parallel processing in terms of DSP, own pipelines and reconfigurable logic.

### What is CNN?
**CNN** - Convolutional Neural Network, also known as ConvNet. It is a class of Deep Neural Networks and consists of a lot of building blocks (for example, normalisation, activation, pooling etc.), and the most expensive operation is Convolution. **Convolution** is an operation on functions, that express, how one modifies the shape of another one. Convolution layers take about 80% of resources.  It is a very complex operation that needs a lot of multiplications. For example, if the input is RGB N*M image, and kernel size is k*k, the time complexity will be O(N\*M\*k\*k).

CNN is trendy in Image and Video processing for recognition, classification and analysis problems. Also can be successfully used for time series analysis/predictions and natural language processing (1D conv). For example, when we need blur, emboss, sharpness or edge detection on the image, convolution can be used in all cases. [[Ref](https://en.wikipedia.org/wiki/Convolution)].
<img src="images/conv.jpg">

#### Why to use FPGA?
- GPU is specified only for matrix multiplication, but FPGA can be used for much more approaches
- TBD

## Solutions
So let us make a fast overview of the most popular approaches to run and accelerate inferring of CNN's on FPGA's

### OpenVINO
Terasic boards are the most popular FPGA's in the world, so if somebody has ever worked with FPGA, most likely he deals with Intel Terasic's chips, such as Cyclone or Stratix. <br>
**OpenVINO** - where VINO stands for Open Visual Inference and Neural network Optimization - a toolkit from Intel to "extend computer vision and non-vision workloads across Intel® hardware, maximizing performance" [[Ref](https://docs.openvinotoolkit.org/latest/index.html)]. It is developed for usage on heterogeneous systems, but only made by Intel®, so when it bought Terasic Inc, OpenVINO extends to supporting FPGA's as well. Personally I have no expertise with this instrument, so more details are in other our article [here](/movidius) and [here](/movidius-2).

### Mipsology
Mipsology product called **Zebra** - a deep learning compute engine for neural network interface [[Ref](https://mipsology.com)]. They promise that if the network was trained in any method, it could be run on CPU, GPU and FPGA with zero efforts without any changes inside the neural network and training process, what is more, it will be faster than before [[Ref](https://www.xilinx.com/video/events/mipsology-demonstrates-zebra.html)].

### Vitis AI
Vitis - SDK from Xilinx. More info [here](https://www.xilinx.com/products/design-tools/vitis.html). <br>
<img src="images/vitis.jpg">
**Vitis AI** is a development platform for AI inference on Xilinx hardware platforms (FPGA, Cloud) [[Ref](https://www.xilinx.com/products/design-tools/vitis/vitis-ai.html)]. It supports models designed and trained on Caffee, TensorFlow, and PyTorch. The workflow is: <br>
- Develop a model using any framework (but using only supporting layers and versions, that are described in the official documentation)
- Train the model 
- Optimize
- Compile 
- Quantize
- And all using Vitis AI tools
- Run the model on the DPU (Deep learning Processing Unit) - a particular IP block for Xilinx boards with UltraScale+ chip on it. It is a part of Vitis AI.<br>
<br>
That is it! The development process is simple and is designed for AI programmers as much as for Embedded programmers.
Performance (from official sites):
Optimizer: reduce the model complexity from 5 to 50 times. Loss: up to 1% of accuracy.
Quantizer: Float32 -> Int8 quantization to make the model faster and lighter. Loss: up to 1% of accuracy.
Performance (from experience): <br>
As far as it is impossible to deploy unsupported layers, by replacing Instance norm with the Batch norm, it managed to reduce the time in average from 32ms to 9ms, but the MAPE (mean absolute percentage error) is up to 10%. We think it is because of replacing such a vital layer but cannot check this assumption.

### PYNQ
PYNQ - open-source project from Xilinx, that makes possible to use Python language and libraries to run it on Xilinx platforms [[Ref](http://www.pynq.io/)]. PYNQ is usually used for developers, who want to make a fast test of their solutions on FPGA without programming on C++. There are eight officially supported boards, but as far as PYNQ is open-sourced, there are much more unofficially supported boards, only by the community. All such boards use DPU,(Deep learning Processor Unit deployed on the FPGA) to process the code.<br>
What is more - about CNN's - there are many examples of running CNN's on FPGA's ([for example here](https://github.com/awai54st/PYNQ-Classification)), but cannot even talk about acceleration - PYNQ is a very high-level approach, easy to use but tough to optimize. So let us look at low-level approaches.

## Approaches for Convolution layers acceleration
Low-level programming is tightly connected with optimization and self-management of all resources - from memory to counting and reducing a real number of operations. <br>
Neural Networks are very heavy - they can be a hundred megabytes. Not every embedded device, especially FPGA, can contain and run such networks, only the best of them. And for what? Embedded devices used to be low-power, high-performance and highly-optimized. So how to accelerate CNN's for FPGA's?

#### GEMM
As it was mentioned before, the most expensive operation in the convolutional networks is the 2d convolution. To accelerate the calculations, we can use GEMM - General Matrix Multiply. The `im2col` operation can be used to make this.  
<img src="images/im2col.png"><br>
After the transformation of the kernels and input image, the convolution operation becomes simple matrixes multiplication, and after that, the repatch is required. There are many ways to accelerate matrixes multiplication.
<img src="images/simd.png">
For example, using CPU's SIMD registers (or even CUDA SIMT - single instruction multiple threads approach) vectors can be multiplied in one tact, and it can accelerate up to two times [[Ref](https://github.com/Myralllka/SOFTSERVE_CNN_convolution_2D)]
GEMM also can be used on the FPGA, and this approach is useful for CUDA, but not the best for the gate arrays.

#### Quantisation
Usually, CNN's weights are floating points, FP32 numbers. It is so to make accuracy as high as possible during training. However, is it necessary in the outgoing network? There are Binary neural networks, and their final weights are booleans, 0 or 1. <br>
FP is vital for training, but not in the outgoing network. Training time is not so important, as final execution accuracy, but add or multiply FP numbers is much slower, needs more power and hardware resources, than INT. <br> 
That is why quantisation is a solution - we can convert FP32 to INT16/INT8 numbers, calculate the accuracy, latency and throughput and compare them. The result can be better if used not just rounding, but deep analysis with it. <br>
Every system has its approach to quantisation, Tensorflow has supervised and unsupervised approaches [[Ref](https://www.tensorflow.org/lite/performance/post_training_quantization)], Caffe has Ristretto  [[Ref](http://lepsucd.com/ristretto-cnn-approximation/)]. Vitis AI has its optimiser in pair with a quantiser. In all cases, we get some increase in latency with a decrease in accuracy. <br>
Important info - it is necessary to convert not only weights, but also layers, operations on that weights.

#### Parallelism
**Parallelism** or **parallel computing** is the approach to programming where one task is divided into a lot of small tasks and compute simultaneously. FPGA is one of the best examples of the hardware realisation of parallelism. As far as it is possible to make our horizontally scalable circuit, it is possible to scale it on the FPGA. Convolution is the operation that can be easily parallelised, as shown here: 
<img src="images/conv_parallel.jpg"><br>
So we can either simultaneously apply multiple kernels to the image or simultaneously apply each kernel in a few positions. In both cases, we have parallelism that can be implemented in hardware using the FPGA.

### Different FPGA design types, caching and pipeline
One of the key challenges in designing high-performance FPGA-based CNN accelerator is to take full advantage of the on-chip computing resource. That is why the DSP, logic, caching and pipes are so important. Convolution, by itself, consists of MAC (multiply-accumulate operations). That is why authors of **DSP-Efficient Hardware Acceleration of Convolutional Neural Network Inference on FPGAs** article divided all possible inferences into three (and we add fourth, thair) groups of approaches.

#### SDConv
"Directly exploit the parallelism of the convolution computation in Spatial Domain by performing a massive number of MAC operations on a large array of DSP blocks in every cycle." It has a computational roof that is proportional to the number of MAC units and the operations frequency.

#### FDConv
Frequency Domain Convolution. "By transforming the data into frequency domain representation, the sliding window operation of SDConv turns into an inner product operation, which significantly reduces the number of the MAC operations required for convolution." Also, the authors presented a highly optimized FDConv, which "saves up to 73% of the MAC operation, and result in a theoretical speedup of 3.7x in peak performance".

#### SpConv
Sparse Convolution - "Scheme which saves MAC operation by directly pruning the CNN model. Unimportant weights (parameters) are forced to zero during the training or fine-tuning stages so that they no longer contribute to any computational workload and memory bandwidth during inference computation."

#### ABM-SpConv
So authors sad that the main problem is that all of this computations can not be achieved in real-life approaches, because they all utilised 97-100% DSP blocks, but in the applied problems such as robotics, autonomous vehicles -  the CNN is not the most important part, there also can be some units that need DSP blocks, so the results of computations can be far away from described. So they introduced a new kind of designs called Accumulate-Before-Multiply Sparse Convolution. The main idea is to decouple the accumulate and multiply operations, create a useful cache using the on-chip memory and create pipelines to make it faster.

##### Caching
the FPGA have much on-chip memory that is uniformly distributed on the board, that can be used as a cache or RAM and makes the latency less than accessing the flash memory. For example, the design of Intel® Cyclon 10
<img src="images/scheme.png"> [[Ref](https://www.intel.co.jp/content/dam/altera-www/global/en_US/documentation/rqk1517250959424/ezd1517849810689.png)] <br>
So the on-chip memory can be used in different ways, and in the case of the ABM-SpConv, they use the memory to cache the results of multiplications and number of each multiplication in specific tables, that helps to accelerate the final inferences.

##### Pipelines
The pipeline is one more powerful feature that is often used for the low-level optimizations on the hardware. Pipelines help to reduce the number of memory accesses by using only on-chip memory and DSP blocks. It is a streaming approach to programs developing.

#### Winograd 
There is a large group of algorithms that make the convolution faster. They are known as **fast convolution** algorithms. The very first algorithm in this group was [Strassen algorithm](https://en.wikipedia.org/wiki/Strassen_algorithm), that reduced the number of operations from ![](https://latex.codecogs.com/svg.latex?O%28n%5E%7B3%7D%29) up to ![](https://latex.codecogs.com/svg.latex?O%28n%5E%7B2.8074%7D%29). The next iteration of researches was an article "Matrix Multiplication via Arithmetic Progressions" by Don CopPersmith and Shmuel Winograd, 1987. In the proposed optimisations to the Strassen's idea, that now known as **Coppersmith–Winograd algorithm**, they describe an approach that reduces the complexity up to ![](https://latex.codecogs.com/svg.latex?O%28n%5E%7B2.375477%7D%29) and this approach becomes the best matrices multiplication approach up to the 2010 year. After that, there were few optimisations, and now the fastest known algorithm complexity is ![](https://latex.codecogs.com/svg.latex?O%28n%5E%7B2.3728639%7D%29), proposed in 2014 by François Le Gall, but this improvement is such small that is not very important.
The main idea behind all these algorithms is to reduce the number of multiplications by using different transformations. For example, one-dim convolution:<br>
f - input image, g - kernel<br>
![](https://latex.codecogs.com/gif.latex?f%20%3D%20%5B1%2C%202%2C%203%2C%204%5D%2C%20g%20%3D%20%5B-1%20-2%2C%20-3%5D)<br>
using `im2com` operation, **f** and **g** will be: <br>
![](https://latex.codecogs.com/gif.latex?f%20%3D%20%5Cbegin%7Bbmatrix%7D%201%2C%202%2C%203%20%5C%5C%202%2C%203%2C%204%20%5Cend%7Bbmatrix%7D)
![](https://latex.codecogs.com/gif.latex?g%20%3D%20%5Cbegin%7Bbmatrix%7D%20-1%20%5C%5C%20-2%5C%5C%20-3%20%5Cend%7Bbmatrix%7D)<br>
![](https://latex.codecogs.com/gif.latex?result%20%3D%20%5Cbegin%7Bbmatrix%7D%201%2C%202%2C%203%5C%5C%202%2C%203%2C%204%20%5Cend%7Bbmatrix%7D%20*%20%5Cbegin%7Bbmatrix%7D%20-1%20%5C%5C%20-2%5C%5C%20-3%20%5Cend%7Bbmatrix%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20m_1%20&plus;%20m_2%20&plus;%20m_3%5C%5C%20m_2%20-%20m_3%20-%20m_4%20%5Cend%7Bbmatrix%7D)<br>
Generalization: <br>
![](https://latex.codecogs.com/gif.latex?result%20%3D%20%5Cbegin%7Bbmatrix%7D%20d_0%2C%20d_1%2C%20d_2%5C%5C%20d_2%2C%20d_3%2C%20d_4%20%5Cend%7Bbmatrix%7D%20*%20%5Cbegin%7Bbmatrix%7D%20g_1%5C%5C%20g_2%5C%5C%20g_3%20%5Cend%7Bbmatrix%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20m_1%20&plus;%20m_2%20&plus;%20m_3%5C%5C%20m_2%20-%20m_3%20-%20m_4%20%5Cend%7Bbmatrix%7D)<br>
where: <br>
![](https://latex.codecogs.com/gif.latex?%5C%5C%5C%5Cm1%20%3D%20%28d_0%20-%20d_2%29g_0%5C%5C%5C%5C%20m_2%20%3D%20%28d_1%20&plus;%20d_2%29%5Cfrac%7Bg_0%20&plus;%20g_1%20&plus;%20g_2%7D%7B2%7D%5C%5C%5C%5C%20m_3%20%3D%20%28d_2%20-%20d_1%29%5Cfrac%7Bg_0%20-%20g_1%20&plus;%20g_2%7D%7B2%7D%5C%5C%5C%5C%20m_4%20%3D%20%28d_1%20-%20d_3%29g_2)<br>
So we need only 4 `mul` operations instead of 6 in traditional matrices multiplication. One more important thing about kernels: <br>
![](https://latex.codecogs.com/gif.latex?%5C%5C%5C%5C%20%5Cfrac%7Bg_0%20&plus;%20g_1%20&plus;%20g_2%7D%7B2%7D%5C%5C%5C%5C%20%5Cfrac%7Bg_0%20-%20g_1%20&plus;%20g_2%7D%7B2%7D%5C%5C%5C%5C)<br>
this multiplier depends only on kernel, so it can be calculated once and be cached.


Details are well described in appendix A [here](https://www.mdpi.com/1999-4893/12/5/112/pdf) for Strassen algorithm, and [here](https://www.sciencedirect.com/science/article/pii/S0747717108800132?via%3Dihub) for Winograd "Matrix Multiplication via Arithmetic Progressions".

### Summary

`TBD`

## Sources
- **MIT Technology Review**, February 24, 2020: "We're not prepared for the end of Moore's Law" by Devid Rotman, [[Ref](https://www.technologyreview.com/2020/02/24/905789/were-not-prepared-for-the-end-of-moores-law/)] <br>
- **Intro to Vitis AI**, [[Ref](https://www.slideshare.net/insideHPC/introducing-the-vitis-unified-software-platform-for-programming-fpgas)]<br>
- **Electronic Engineering Journal**, October 4, 2019: "Xilinx Vitis and Vitis AI Software Development Platforms" by Kevin Morris [[Ref](https://www.eejournal.com/article/xilinx-vitis-and-vitis-ai-software-development-platforms/)]<br>
- **A Parallel FPGA Implementation of Image Convolution**, 2016 by Henrik Ström [[Ref](https://www.diva-portal.org/smash/get/diva2:930724/FULLTEXT01.pdf)]<br>
- **DSP-Efficient Hardware Acceleration of Convolutional Neural Network Inference on FPGAs**, September 2019, Dong Wang, Ke Xu, Jingning Guo, and Soheil Ghiasi<br>
- **A framework for generating high throughput CNN implementations on FPGAs,** by H. Zeng et al., in Proc. FPGA, New York, NY, USA, 2018, pp.117-126.<br>
- **A Scalable Framework for Acceleration of CNN Training on Deeply-Pipelined FPGA Clusters with Weight and Workload Balancing**, January 2019, by Tong Geng, Boston University, [[Ref](https://www.groundai.com/project/a-scalable-framework-for-acceleration-of-cnn-training-on-deeply-pipelined-fpga-clusters-with-weight-and-workload-balancing/1)]<br>
- **FPDeep: Acceleration and Load Balancing of CNN Training on FPGA Clusters**, by Tong Geng, Tianqi Wang, Ahmed Sanaullah, Chen Yang, Rui Xu, Rushi Patel, Martin Herbordt. [[Ref](https://www.bu.edu/caadlab/FCCM18a.pdf)]<br>
- **Convolution Accelerator Designs Using Fast Algorithms**, May 2019, by Yulin Zhao, Donghui Wang, Leiou Wang<br>
- **Matrix Multiplication via Arithmetic Progressions**, May 1987, by Don CopPersmith and Shmuel Winograd<br>
- **Understanding Winograd Fast Convolution**, Jul 2019, by Deepak Mangla. [[Ref](https://blog.usejournal.com/understanding-winograd-fast-convolution-a75458744ff)]
