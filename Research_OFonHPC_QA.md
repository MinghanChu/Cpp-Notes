# OpenFoam postprocessing (on HPC)

[toc]

### Introduction

The most prominent feature on HPC is parallel computing. Parallel computing greatly reduces the time taken for numerical computations. However, there are always trade-offs associated with it. Here are the costs I have noticed after numerous tests:

1. **Not stable** OpenFoam provides several ways to decompose the single computational zone into number of parts between which processors must communicate with each other. These methods include: `simple`, `hierarchy`, `scotch`, etc. Right now I am sticking with `scotch` to let the algorithm behind split the zone for me. You could also try the other two, however you should be very careful to specify the coefficients accompanied with them. As if not proper values are used, decomposition will fail. 
2. **Prefer more cpus on HPC** It is weird that HPC, cedar or graham, intends to have users use more cpus to run their jobs rather than very few number of cpus, e.g. 2 or 4. For example, I got error messages popped up relating to mpi when I gave 2 cpus to the `UQT3A` case. This error went away once I increased the number of cpus to 32. 



#### Postprocessing

Recall **in OpenFoam** **applications** are split into two main categories: **solvers** and **utilities**. They are executable binary data, known as executables. Except those two, another important concept in OpenFoam is **libraries.** Why is library important? My own UQ model, **MyUQ.C/H** files, are compiled into shared libraries, with extension`.so`. When being executed through **UQsimpleFoam solver**, dynamic linker will search the needed functions when needed in `.so` libraries. 

**!!! There is actually not a clear difference between library and executables**

```
1.
WC:
你发的那段话只强调了 entry point， 通俗说是 main function。但即便是这一点，也不是绝对的，譬如在interpreted language，像bash，就没有 main function。而bash script 本身既可以作为library （譬如在重新编译时，负责清理上一次编译过程中产生的中间文件） 也可以作为 executable（只是为了清理文件）。
此外，常见的区别executable和所谓library的还有linking，binary vs source code 等。现在很多library自己就是executable，在python里面很常见，主要是为了提供不同的使用方法。

2.
WC:
We should reiterate from the outset that OpenFOAM is a C++ library used primarily to create executables, known as applications. OpenFOAM is distributed with a large set of precompiled applications but users also have the freedom to create their own or modify existing ones. Applications are split into two main categories:

显然作者强调了，在他的语境下，application 就是 executable
“executables, known as applications. ”

3.
MC: 那么伟成，我这样问你一个问题：我们为什么要有library？
WC: 为了reuse code

4.
MC: 但executable只能specific？

WC: 不一定，executable可以把接口都做成输入的参数，你的 bash command就是这样的

5.
MC: 那是什么让他还能叫做exe呢？他一定和lib在某些方面有重要不同是吧？
WC: 区分是相对的不是绝对的

6.
MC: 在openfoam里，solver 和utilities都是成品，而turbulence models是输入后处理数据的，所以of叫applications

WC: 你这么理解没啥问题

```



Based on the above conversation: **In OpenFoam environment**, utilities (even though compiled into .so files) are called **applications**, because they are used to process data, e.g. postprocessing. To do post-processing, users need to give corresponding `functionObjects` which are shared libraries to be used by dynamic linker.



##### Exactly how to perform post-processing?

As the name explains post-processing happens after calculations are finished. However in OpenFoam you can let post-processing work after each time step while simulations are running. 

On HPC, **parallel computing** seems not stable working with the post-processing taking place at each time-step as the computational domain is split into several regions, therefore we would like to perform it after the whole job is complete.

**Post-processing** commands in OpenFoam:

**`UQsimpleFoam -postProcess -func wallShearStress`**

Note the above commands includes the solver, i.e. **`UQsimpleFoam`**, for calculation of **wall shear stress**. Because it requires the information from it. There are some quantities that do not need to include the solver, e.g.

 **`postProcess -func wallShearStressGraph`.** 

So pay attention to it. 

In order to successfully execute the command above, you also need to specify which **`functionObjects`** , which is the already compiled library (application), the command is looking for. In this particular example, the functionObject named **`wallShearStress`** is selected.

