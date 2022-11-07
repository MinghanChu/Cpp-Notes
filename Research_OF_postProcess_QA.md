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



#### force coefficients

`````
Option            | Description
none              | Trigger is disabled
timeStep          | Trigger every 'Interval' time-steps, e.g. every x time steps
writeTime         | Trigger every 'Interval' output times, i.e. alongside standard field output
runTime           | Trigger every 'Interval' run time period, e.g. every x seconds of calculation time
adjustableRunTime | Currently identical to "runTime"
clockTime         | Trigger every 'Interval' clock time period
cpuTime           | Trigger every 'Interval' CPU time period
onEnd             | Trigger on end of simulation run

executeControl: when the object is updated (for updating calculations or for management tasks),
writeControl: when the object output is written (for writing the calculated data to disk).
`````



````c++
//Note: "writeControl" might overlap the results from "executeControl", e.g. "executeControl timeStep" you will definitely 
//have overlaps with time steps from "writeControl" 

//RECOMMENDATIONS: 
		//for small "deltaT" you may want to set "writeControl writeTime" instead of "writeControl timeStep", where the latter 
		//will print too much data and hard to observe

//Keep in mind: on default "executeControl timeStep", meaning QoIs at every time step will be printed out if you do not
//deliberately set "executeControl"

//OBSERVATIONS:
//for force coefficients, i.e. Cd and Cl, even if you have set "writeControl writeTime", you will still have them printted
//out every time step in the postprocessing folder. While for "yPlus" and "wallShearStress" for example, unless you set
//"writeControl timeStep" you will only have them outputted at specified "writeInterval". In addition, you will also have

	forceCoeffs //name can be anything
	{
		type 		forceCoeffs;//must be "forceCoeffs" (c++ type)
		libs		("libforces.so"); //the utility (library) must be "libforces.so" for v1812
		
    //My understanding:
    //writeControl
    		//writeTime: output time/iteration interval specified by the user, i.e. "writeInterval"
    		//timeStep: smallest time interval the simulation is proceeding forward, i.e. "deltaT"
    
    //writeInterval: Steps/time between write phases, depending on if "writeTime (writeInterval)" or "timeStep (deltaT)"
    
    //NOTE: No matter if "executeControl" is triggered, as long as "writeControl" is specified (on default "timeStep"), QoIs 
		//(here forceCoeffs) will be computed. On the other hand, the purpose of "executeControl" is just used to add additional
    //control for outputting that QoIs. You do not need "executeControl" unless you need to watch closely what is happening at
    //particular time steps. 
    
		timeStart		20;//also controls when to output in postprocessing folder
    timeEnd			25;
    
		writeControl	writeTime; //on default "timeStep": when "timeStep" is used you will have this particular field outputted 
    												 //at every time step, meaning tons of data outputted!!!!!!!!!!!!
		writeInterval	1; //on default "1"
	
    enabled	true;//true means printing out coefficients and false means skip this utility (no coefficients will be printted out)

		log		no;//show results on GNU

		//timeStart	0;
		//timeEnd		1000;

    //executeControl calculates/updates the quantities of interest
    		//independent of “writeControl” which is dedicated to outputing results, while "executeControl" is for actual calculating
    		//of the QoIs
    		//executeInterval means n * executeControl (timeStep or writeTime) where n is the number of intervals you want QoIs to be
        //computed 
		executeControl  writeTime; //on default "timeStep"
		executeInterval 1; //on default "1"
		
		patches
		(
			Airfoil
		);
	
		rho		rhoInf;	
		rhoInf		1.2;
		CofR		(0 0 0);
		pitchAxis	(0 1 0);
		liftDir		(-0.139173 0.990268 0.00000);
		dragDir		(0.990268 0.139173 0.00000);
		magUInf		0.9;
		lRef		1;
		Aref		0.1;
	}
````







