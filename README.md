# GPU-FPX
A tool to detect the exceptions on the device at run time.

GPU-FPX is an extension of [BinFPE](https://github.com/LLNL/BinFPE), which is a too based on [NVBit](https://github.com/NVlabs/NVBit). 

 The full source release is subject to license clearance and done by Phase-2.

# Artifact appendix
### Smmary
Our artifact GPU-FPX conducts binary-instrumentation based analysis of floating-point exceptions such as division-by-zero in CUDA programs that employ numerical (floating-point) calculations. It employs NVIDIA's NVBit-based instrumentation.
		
__`Input`__: The input to GPU-FPX is a CUDA program with its test-benches.
		
__`Output`__: The output of GPU-FPX is a report that presents the number of exceptions, the location information of exceptions, the number of launched kernels, and the number of floating-point instructions.  
		
__`Method`__: Upon paper acceptance, we will provide a Docker container that will download NVBit (which we cannot redistribute) and patch-in the necessary extensions to build GPU-FPX. The artifact evaluator must have access to a GPU-enabled machine. Our scripts will run 21 programs (those presented in our exceptional table) with GPU-FPX automatically and generate the exceptional report and evaluated time file. 

We now detail these steps in the rest of this document.
	
GPU-FPX is aimed at detecting exceptions on devices, which can help the designers debug. We have evaluated GPU-FPX on a total of 115 GPU programs. Here we will demonstrate how to use GPU-FPX and how we are running all the programs in our paper. We embed the whole 115 programs in our docker container but also will provide a script to run a small number of benchmarks for AD evaluators. 
	
	
### GPU-FPX
GPU-FPX is binary-instrumentation tool based on NVIDIA's NVBit. Thus, to build our tool, you need to first download [NVBit](https://github.com/NVlabs/NVBit/releases) and then put the source code of GPU-FPX in the `tools` directory of NVBit repo.

At this point, you must go to the GPU-FPX folder and run `make` to build our tool as the shared library `GPU-FPX.so`. It is then very easy to inject  `GPU-FPX.so` to the program simply through `LD_PRELOAD` as
```
		LD_PRELOAD=GPU-FPX.so ./program_to_evaluate
```
The minimum configuration to build GPU-FPX obeys the requirement of NVBit: 
- SM compute capability: >= 3.5 \&\& <= 8.6
- Host CPU: x86\_64, ppc64le, arm64
- OS: Linux
- GCC version: >= 5.3.0
- CUDA version: >= 8.0 \&\& <= 11.x
- CUDA driver version: <= 495.xx
- nvcc version for tool compilation >= 10.2
	
We have built our tool in two machines, and the configurations are:
	
__Machine 1__:
1. an AMD Ryzen 5 3600 6-Core processor;
2. a NVIDIA GeForce RTX 2070 SUPER GPU (Turing architecture and 8 GB memory);
3. the Ubuntu 20.04.3 LTS system;
We compile GPU-FPX on machine 1 using the CUDA-11.5 toolkit. 
__Machine 2__:
1. Intel(R) Core(TM) i9-10900K CPU (10 cores) @ 3.79 GHz;
2. a NVIDIA GeForce RTX 3060 GPU (Ampere architecture and 12 GB memory);
3. the Ubuntu 20.04.4 LTS system;
4. We compile GPU-FPX on machine 2 using the CUDA-11.6 toolkit. 
	
For simplicity, we will refer to machine 1 and machine 2 when talking about the hardware settings. 
### FPXBench
#### Overview
We collected 28 programs from __Lonestar__(bh), __Polybench__(corr, gramschm), __SHOC__(fft, md, s3d, segemm), __GPGPU-sim__(libor, lps, rayTracing, wp), __Parboil__ (cutcp, mri-gridding, mri-q, spmv, stencil, tpacf), __Rodinia__(backprop, cfd, gausssian, heartwall, hotspot, kemans, lud, myocyte, nn, srad) and a MRI application from~\cite{LI2015998}. 
#### Software
FPXBench is simulated on machine 1. 
And the software we are using are:
1. `NVCC` from CUDA-9.2 toolkit, with flags `-arch=sm\_70` and `--generate-line-info` (if you want to get the line information)
2. `gcc-7` installed with `apt-get install gcc-7`
3. `Python 3.8.10`

`Python3` here is used to analyze the report and generate the table presented in the paper. 
	
#### Experiments
The experiments we perform on this suit are
1. Using the default running commands in these programs, we injected GPU-FPX and reported the exceptions detected in each program. At the same time, we used the `time` command to record the `real` time for running each benchmark.   
2. We then add the flag `--use_fast_math`, and compile each program. Also, we record exceptional reports. 
3. In this experiment, we created two specified NVBit tools: one removes the channel transformation on the device, and one removes both channel transformation and checking procedure on the device. We applied GPU-FPX and these two specified tools to each program and recorded the running time. 

We have three bash scripts to run the above three experiments automatically and export the data to text files. Additionally, we have three python scripts to generate the exceptional table, `fast_math opt.` table, and the bar diagram in our paper. 
	
	\section{CUDA-Sample}
	\paragraph{Overview} There are more than 100 programs in the official Cuda-Samples\footnote{https://github.com/NVIDIA/cuda-samples.git}. However, we removed programs
	\begin{itemize}
		\item requiring more than one GPU;
		\item requiring display;
		\item without any FP instructions.
	\end{itemize} 
	Finally, we have 77 programs to evaluate.
	\paragraph{Software} Cuda-Samples are executed on machine 2. We are using
	\begin{enumerate}
		\item {\tt NVCC} from CUDA-11.6 toolkit;
		\item {\tt gcc 9.4.0}; 
		\item {\tt Python 3.8.10}.
	\end{enumerate} 
	Notice that we are using the default compilers' flags. 
	\paragraph{Experiments}
	We perform two experiments on this collection.
	\begin{enumerate}
		\item We recorded the exceptional report for each program.
		\item  We ran these benchmarks with device-checking and host-checking separately with 1500 seconds time limit. Also, recorded each exceptional report and running time of both approaches.
	\end{enumerate}
	The above experiments could be executed using one bash script. We then use these information to generate exceptional table and performance scatter plotting using python scripts.  
	\section{NPB-GPU: }
	\paragraph{Overview}  This test suit contains 8 programs and each accept 8 classes {\tt S, W, A, B, C, D, E, F} with the ascending data size. In our experiment, we evaluated each program with three classes {\tt S, W, A}.
	\paragraph{Software} We run NPB-GPU on machine 1, and related software versions are
	\begin{enumerate}
		\item {\tt NVCC} from CUDA-11.5 toolkit, with flag {\tt arch=sm\_70}. 
		\item {\tt gcc 9.4.0}
		\item {\tt Python 3.8.10}
	\end{enumerate}
	\paragraph{Experiment}
	We compiled each benchmark with three classes with a bash script and then ran these 24 programs with GPU-FPX using another bash script. Later, we used the a python script to extract the exceptions for each case. It turns out there's no exceptions in the 24 executables.  
	\section{CuMF-Movielens: } We built this project as its {\tt README.md} instructed on machine 1. Here the software to build this application are 
	\begin{enumerate}
		\item CUDA-9.2 toolkit;
		\item {\tt gcc-7};
		\item {Python 2.7.18} and {\tt ipython} package.
	\end{enumerate}
	The {\tt python2} here is used to download and preprocess the \textit{Movieles10M} data. After compiling the programs, we ran the default command with GPU-FPX and recorded the report.  
	\section{HPCG: } There are several versions including a docker container. Here we chose the \textit{HPCG 3.1 Binary for NVIDIA GPUs Including Ampere based on CUDA 11}, and run it on machine 2. Since it is a binary, we only care about the CUDA toolkit version. Here we used CUDA-11.6. 
	
	In the experiment, we evaluated eight different data grid settings provided by NVIDIA in \textit{HPCG 3.1 Binary for NVIDIA GPUs Including Volta based on CUDA 10}. The exceptional report is generated in each data grid. We later extracted the exceptional information using a python script. 


# License 
`GPU-FPX` is distributed under the terms of the MIT license - the same license as `BinFPE`.

