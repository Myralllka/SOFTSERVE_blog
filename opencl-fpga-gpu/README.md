# OpenCL Performance on FPGA and GPU

The given article is continuation of our [FPGA research](README.md).

There are common programing models for FPGA. The traditional way, also called RTL-design, requires implementation of algorithms in LDH language such as Verilog or VHDL. Another approach is to use higher level language such as OpenCL to implement accelerated function (kernel), and compile it to target hardware (CPU, GPU, or FPGA). OpenCL allows the use of a C-based programming language for developing code across different platforms. OpenCL is a portable, open, royalty-free standard.

OpenCL is currently supported by GPU (NVidia, AMD) and FPGA (Altera, Xilinx) vendors. In particular, Xilinx supports OpenCL development via SDAccel™ development environment, which is also supported in AWS cloud.

With SDAccel™ and Amazon AWS F1 instance it is possible to design, test and run custom developed algorithms on FPGA in cloud.

[Previously](https://github.com/softserveinc-rnd/fpga-gpu-benchmarking/blob/master/README.md) we compared FPGA vs GPU performance on Neural Network inference. GPU won in performance per dollar on AWS with F1 instance in most tests.

What about other applications, for instance unsipervised learning, computer vision?
We took 2 algorithms: [naive k-nn clustering](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm), and [edge detection](https://en.wikipedia.org/wiki/Sobel_operator) and tested it's accelerated implementation. 


# k-means clustering

We used Rodinia-based [benchmark](http://www.cs.virginia.edu/~skadron/wiki/rodinia/index.php/Rodinia:Accelerating_Compute-Intensive_Applications_with_Acceleratros). FPGA implementation is customized code for Xilinx OpenCL Devices. 

Clustering of 1,000,000 records with 34 features to 5 classes. Time is averaged across 10 runs.

OpenCL is used on both platforms for kernel implementation and same code compiled to GPU and FPGA.

| Hardware  | Device                    | Total time, sec  | Power Consumption, Watt | Tx / Watt |
| --------- | ------------------------- | ----------------:| ----------------------: | ---------:|
| FPGA      | Xilinx UltraScale+™ VU9P  |              5.6 |          40 (estimated) |      4460 |
| GPU       | NVIDIA® Tesla® K80        |              3.9 |                     300 |      850  |

# edge detection

Edge detection sobel [algorithm](https://en.wikipedia.org/wiki/Sobel_operator) was tested on 1024x1024 images. For this task, GPU implementation is CUDA-based. Execition time was measured to process 1,000 images

| Hardware  | Device                    | Total time, sec    | Power Consumption, Watt | Tx / Watt |
| --------- | ------------------------- | ------------------:| ----------------------: | ---------:|
| FPGA      | Xilinx UltraScale+™ VU9P  |              0.330 |          40 (estimated) |        75 |
| GPU       | NVIDIA® Tesla® K80        |              0.320 |                     300 |        10 |

# Conclusion

In the given experiment GPU slightly outperforms FPGA in absolute speed but FPGA is 5x ~ 7x better than GPU in transactions / watt.

Author: Oleksandr Sukholeyster
