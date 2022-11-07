# Solving steady and unsteady problems in OpenFoam

[TOC]

**keywords and logic that help understanding**

bounded ->stable ->diffusive/inaccurate

unbounded ->unstable/oscillatory->not diffusive/accurate







### steady solver - simpleFoam

After reviewing the **solver study** I conducted in 2019, found that there are several **key parameters** I was focus on tuning, and studied their influences on final results:

#### **fvSolution** (how to **solve the linear system** of discrete algebraic equations)

1.  under **solvers** what solver should be chosen for `"(U|k|omega|gammaInt|ReThetat)"`

   Choose between `PBiCGStab` and `smoothSolver`(most of times I used `smoothSolver` in my tests)

2. under **SIMPLE** what value should be assigned to`nNonOrthogonalCorrectors` , e.g. 0-2. 

   I found value of 2 resulted in bad residuals. I set it be 1 or 0 in my tests.

    

#### **fvSchemes** (how to **discretize** in space each term of governing equations, e.g. diffusive, convective, gradient, and source terms)

##### ==ddtSchemes==

1. under **ddtSchemes** (time/unsteady term discretization schemes)

   `$WM_PROJECT_DIR/src/finiteVolume/finiteVolume/ddtSchemes`

   * **steadyState** for steady state simulations (**implicit/explicit**)

   * **Euler** time dependent **first order** (**implicit/explicit**), **bounded** (**first order methods** are **bounded** and **stable**, however **diffusive** (keep **CFL < 1** then **diffusion becomes less significant!**.)

   * **backward** time dependent **second order** (**implicit**), **bounded/unbounded** (**second order methods** are **accurate**, however **oscillatory** (still people always want accurate solution!))

   * **crankNicolson** time dependent second order (**implicit**), **bounded/unbounded** (uses a **blending factor**)

     ```c++
     ddtSchemes
     {
     	default		CrankNicolson x; 
     }
     ```

     **0 <=x <= 1**: 0 means pure **Euler** (robust but **first-order** accurate); 1 means pure **Crank-Nicolson** (**second-order** accurate but oscillatory). **Smart option:** to set the blending factor to **0.5** (results in between first and second order accuracy, get best of both worlds!). **x = 0.9!** is safe to use for most applications.

     

##### ==divSchemes==

2. under **divSchemes** (convective terms discretization schemes)

   `$WM_PROJECT_DIR/src/finiteVolume/interpolation/surfaceInterpolation`

   * **upwind** for **first-order** accurate ,**bounded** (stable but diffusive)

   * **linearUpwind** for **second-order** accurate, **unbounded** (accurate but oscillatory, but we always want a second-order accurate!)

     

     when you use **linearUpwind** for `div(phi,U)`, you need also specify the **gradSchemes** for `grad(U)`, same applies for **scalars:** **k, epsilon, omega, T**. (This must be related to the math behind **linearUpwind**)

     ```c++
     gradSchemes
     {
       default  Gauss linear;//all rest scalars are treated using Gauss linear scheme to compute gradients
     	
     	grad(U)  cellMDLimited Gauss linear 1.0; //special treatment for U
     	
     }
     
     divSchemes
     {
        default             none;
     
         div(phi,U)          bounded Gauss linearUpwind grad(U);
     
         div(phi,k)          bounded Gauss linearUpwind grad(k);
         div(phi,omega)      bounded Gauss linearUpwind grad(omega);
         div(phi,gammaInt)   bounded Gauss linearUpwind grad(gammaInt);
         div(phi,ReThetat)   bounded Gauss linearUpwind grad(ReThetat);
     
         div((nuEff*dev2(T(grad(U))))) Gauss linear;
     
     }
     ```

     

   * **linear** for **second-order** accurate, **bounded**

   * **vanLeer** for TVD, **second-order** accurate

   * **limitedLinear** for **second-order** accurate, **unbounded**, but more **stable** than pure linear. (recommended for **LES** simulations)



##### ==gradSchemes==

3. under **gradSchemes** (gradient discretization) choose between `cellLimited Gauss linear`, e.g. 0.5 or 1, and `Gauss linear`

   `$WM_PROJECT_DIR/src/finiteVolume/finiteVolume/gradSchemes`

   `$WM_PROJECT_DIR/src/finiteVolume/finiteVolume/gradSchemes/limitedGradSchemes`

   

   * **Gauss** (may use **gradient limiters** , increasing **stability** but adding **diffusion** due to **clipping**, hence **less accurate** )
   * **leastSquares** (may use **gradient limiters**, increasing **stability** but adding **diffusion** due to **clipping**, hence **less accurate**)

   

   **Gradient limiters** are used to **avoid** **undershoots** or **overshoots**:

   

   + **cellMDLimited** (multi-dimensionally limits cell-to-cell values), at least **2nd-order accurate** 												**least diffusive**

   + **cellLimited** (limits cell-to-cell values), at least **2nd-order accurate**

   + **faceMDLImited** (multi-dimensionally limits face-to-face values), at least **2nd-order accurate**

   + **faceLimited** (limits face-to-face values), at least **2nd-order accurate**                                                                                      **most diffusive**

     The above standard limiters will apply to **all components** of the gradient. The **default method** is the **Minmod.**

```c++
gradSchemes
{
	default  cellMDLimited Gauss linear x; 
	grad(U)  cellMDLimited Gauss linear 1.0; //special treatment for U
	
}
```

**0 <=x <= 1**: 0 means **turning off** the gradient limiter (**accurate but unbounded/not stable**); 1 means gradient limiter is **always on (less accurate but bounded/stable) ** . **Smart option:** to set the blending factor to **0.5** (the best of both worlds).



##### ==laplacianSchemes== (note choosing blending factor depends on the ==mesh quality==)

4. under **laplacianSchemes** (Laplacian terms discretization, e.g. diffusion)

   `$WM_PROJECT_DIR/src/finiteVolume/finiteVolume/snGradSchemes`

   + **orthogonal** for a **perfect mesh** (hexahedral meshes with no grading), **second-order** accurate, **bounded** on **perfect meshes**, **without non-orthogonal** corrections
   + **corrected** for meshes **with grading and non-orthogonality**, **second-order** accurate, **bounded** (depending on the mesh quality), **with non-orthogonal corrections**
   + **limited** for meshes **with grading and non-orthogonality**, **second-order** accurate, **bounded** (depending on the mesh quality), **with non-orthogonal corrections**
   + **uncorrected** for **bad** meshes **with grading and non-orthogonality**, **second-order** accurate, **without non-orthogonal corrections**, **stable but more diffusive** than **corrected and limited** methods.

   

   ==I have a sense== that **orthogonal corrections** can result in **less stable** but **more accurate** results. It is unlike the **limiters** for gradients, where the use of limiters **compromise** on **accuracy**. As long as **uncorrected method** is somehow used (including the **limited** that blends the **corrected and uncorrected** methods), you give up accuracy **to some degree**. ==Unless your mesh is bad, always use **corrected**, or generate high-quality mesh first!==

   

   ```c++
   laplacianSchemes
   {
   	default    Gauss linear limited (corrected) x; //"Gauss" is the only option; "linear" is the interpolation method of the 
     																					//diffusion coefficient; "corrected" scheme is on default you can choose to put or 
     																					//not
   }
   ```

   **0 <=x <= 1**: 0 means **uncorrected method**, meaning **less accurate but more stable/bounded**;  1 means **corrected method**, meaning **accurate but less stable/unbounded** is **always on (less accurate but bounded/stable) ** . **Smart option:** to set the blending factor to **0.5** (the best of both worlds). In this case **non-orthogonal** contribution does **NOT** exceed the **orthogonal** contribution. You give up **accuracy** but gain **stability.**

   

   **some useful tips**

   + For meshes with **non-orthogonality < 75 (good quality mesh)**, set **x = 1**;
   + For meshes with **85 > non-orthogonality > 75  (somehow good quality mesh)**, set **x = 0.5**;
   + For meshes **non-orthogonality > 85 (bad quality mesh)**, get a **better mesh!!**, or set **x = 0.333**, and **increase** the number of **non-orthogonal corrections** ;
   + If **LES** or **DES** is used, always set **x = 1**, meaning you MUST get a good quality mesh first!!

   

##### snGradSchemes (note choosing blending factor depends on the ==mesh quality==)

Usually (not necessarily!) use the **same method** as that used for **laplacian terms** (for my research keep them same all the time!)

```c++
laplacianSchemes
{
	default    Gauss linear limited (corrected) x; 
  //"Gauss" is the only option; "linear" is the interpolation method of the diffusion   			 																	 //coefficient
}

snGradSchemes
{
	default    corrected or limited (corrected) x; //in consistent with the laplacian method
}
```





#### What method should I use?

##### Case 1 - fully bounded (most conservative/stable but not accurate)

+ **2nd order** accurate and **fully bounded**
+ **depending on your mesh quality** tune the **blending factor** of **laplacianSchemes** and **snGradSchemes**
+ keep temporal diffusion to a minimum, use **CFL < 2** (or CFL < 1 for Euler then diffusion becomes significant)
+ if during simulation turbulence quantities become **unbounded**, then safely use discretization scheme to **upwind**. After all, turbulence is diffusion.

```c++
ddtSchemes
{
	default		CrankNicolson 0; // "0" is equivalent to "Euler" and "1" is "CrankNicolson"
}

gradSchemes
{
  default   cellLimited Gauss linear 1;//compared to "Gauss linear";
  grad(U)		cellLimited Gauss linear 1;
}

divSchemes
{
  default  none;
  div(phi, U)			(bounded) Gauss linearUpwind grad(U) //compared to "grad" (default scheme is used: see above "gradSchemes")
  div(phi, omega) (bounded) Gauss linearUpwind default; //compared to "Gauss linearUpwind grad"
  div(phi, k)			(bounded) Gauss linearUpwind default; //compared to "Gauss linearUpwind grad"
  div((nuEff*dev(T(grad(U)))))  Gauss linear; 
}
//note under "divSchemes", on default "bounded" (stable but less accurate)
laplacianSchemes
{
  default		Gauss linear limited (corrected) 1; //on default "corrected" scheme is used or equivalent to "limited (corrected) 1"
}

interpolationSchemes
{
  default		linear; //should always linear for my research
}

snGradSchemes
{
  default	corrected or limited (corrected) 1; //always in consistent with "laplacianSchemes"
}
```



##### Case 2 - A very accurate/not diffusive but unstable/oscillatory 

+ use this scheme for LES or DNS when no complex physics are involved, with very high-quality mesh

```c++
ddtSchemes
{
	default		backward; // unbounded accurate/not diffusive but unstable
}

gradSchemes
{
  default   Gauss leastSquares;//may use limiters ondefault no limiter is used, meaning most accurate 
}

divSchemes
{
  default  none;
  div(phi, U)			(bounded) Gauss linear 
  div(phi, omega) (bounded) Gauss linear; 
  div(phi, k)			(bounded) Gauss linear;
  div((nuEff*dev(T(grad(U)))))  Gauss linear; 
}
//note under "divSchemes", on default "bounded" (stable but less accurate)
laplacianSchemes
{
  default		Gauss linear limited (corrected) 1; //on default "corrected" scheme is used or equivalent to "limited (corrected) 1"
}

interpolationSchemes
{
  default		linear; //should always linear for my research
}

snGradSchemes
{
  default	corrected or limited (corrected) 1; //usually in consistent with "laplacianSchemes"
}
```



##### Case 3 - A very stable but too diffusive/inaccurate 

+ start robustly, end with accuracy

```c++
ddtSchemes
{
	default		Euler; // bounded inaccurate/diffusive but stable/not oscillatory
}

gradSchemes
{
  default   cellLimited Gauss linear 1;//limiter is on full power (= 1), meaning most very stable but too diffusive/inaccurate 
	grad(U)   cellLimited Gauss linear 1;
}

divSchemes
{
  default  none;
  div(phi, U)			(bounded) Gauss upwind; //first order bounded stable but inaccurate (upwind "u" lower case!) no grad needed!
  div(phi, omega) (bounded) Gauss upwind; 
  div(phi, k)			(bounded) Gauss upwind;
  div((nuEff*dev(T(grad(U)))))  Gauss linear; 
}
//note under "divSchemes", on default "bounded" (stable but less accurate)
laplacianSchemes
{
  default		Gauss linear limited (corrected) 0.5; 
  //on default "corrected" scheme is used or equivalent to "limited (corrected) 0.5". 0.5 means "corrected" is not on full power
  //hence less accurate. Unlike limiters, "corrected" helps improve accuracy.
}

interpolationSchemes
{
  default		linear; //should always linear for my research
}

snGradSchemes
{
  default	corrected or limited (corrected) 0.5; //usually in consistent with "laplacianSchemes"
}
```









#### **Courant**–Friedrichs–Lewy (CFL) condition 

##### ==controdict==

+ CFL = u $\Delta$ t/$\Delta$ x < $CFL_{max}$, **measures how much information u carries across a cell distance, usually smaller value is preferred**
+  proper CFL value ensures stability and hence **convergence**
+ from the above equation, CFL can be changed by either changing **mesh cell size** or **time step** (easiest way!)
+ "**deltaT**" controls the time size
+ "**adjustTimeStep**" (yes/no) works together with "**maxCo**" (max CFL allowable) and "**maxDeltaT**" (max time step allowable) ; automatically adjust the time step to "maxCo" and "maxDeltaT"; start with "**small** time step"
+ "**runTimeModifiable**" ensures change on-the-fly











#### Linear solvers in OpenFoam (iterative or direct method)

##### ==fvSolution==



###### **What is preconditioner?**

In mathematics, **preconditioning** eases the way of numerically solving algebraic equations of a matrix through transformation into the form that reduces the **condition number** of a problem. In short, it transforms a problem into a form that is more suitable for numerical solving methods. The preconditioned problem is usually solved by an **iterative method**. Selection of the tolerance is of paramount importance (if not solved accurately enough or reaching low tolerance the physics might be wrong). 

**Linear solvers** distinguish between **symmetric matrices (p field)** and **asymmetric matrices (U field)**. Therefore, choosing the right solver according to the particular flow field you are dealing with.



###### **How to define accuracy?**

Since the **linear solvers** are iterative, **residual** should decrease with **number of iterations**. **Residual** is re-evaluated after each iteration. We say we are getting **accurate results** when **residuals** get smaller. That means even if we are getting **accurate results** and it does not necessarily mean we are getting **real (right) results** that are close to what **real physics** is.



+ **smaller time step** helps maintain **stability**, therefore if solution **not converging** **reduce** the time step for the **first iterations**

  

*The solver stops if either of the following conditions are reached:*

**tolerance**: level at which the residual is small enough then we can say we are getting **accurate results**

**relTol**: relative tolerance - the **ratio** of current to initial residues

**maxIter**: maximum number of iterations, **default value:** 1000

(By the way**minIter:** minimum number of iterations, **default value:** 0)



**==using renumberMesh -overwrite==** before running the simulation!!! which dramatically increases the speed of the linear solvers. 



###### **What are the linear solvers in OpenFoam?**

+ GAMG -> multigrid solver (most of times using the GAMG method for **symmetric matrices**, e.g. pressure, should converge **less than 20 iterations**, otherwise using a **smoother**, e.g. **GaussSeidel**)

  + types of smoothers: DIC, DICGaussSeidel, DILU, DILUGaussSeidel, FDIC, GaussSeidel, symGaussSeidel, nonBlockingGaussSeidel

  + using GAMG when you are dealing with large number of processors, e.g. > 500

    

+ PBiCG -> Newton-Krylov solver (need a preconditioner)

+ PBiCGStab -> Newton-Krylov solver (for **asymmetric matrices **，need a preconditioner, e.g. **DILU**. Change the preconditioner if **crashed** or try the **smoothSolver**, usually **faster** than smoothSolver)

  + **asymmetric matrices:** **U** velocity field, and the transported quantities, **k, omega, epsilon, T, and so on**

  + **low cost**, therefore use **tolerance = 1e-8**

  + types of preconditioners: DIC, DILU, FDIC, GAMG, diagonal, noPreconditioner

    

+ PCG -> Newton-Krylov solver (for **symmetric matrices**, need a **preconditioner**, if using **GAMG** is unstable or take too long, choose **PCG**)

+ smoothSolver -> Smooth solver

+ diagonalSolver

  + used for **back-subsitution**, e.g. calculating density using equations **P and T**



###### How to set tolerances?

+ pressure (P) equation is particularly important!! And need to solve accurately. High cost!! 
+ ==start with: **tolerance = 1e-6** and **relTol = 0.01**. After a while, **tolerance = 1e-6** and **relTol = 0**.==
+ ==If the solver is too slow, **tolerance = 1e-4** and **relTol = 0.05** during the first iterations.==





###### pressure (p) - symmetric matrix (using symmetric solvers) and expensive cost, rather important!

```c++
solvers
{
	p
	{
		solver				   PCG//generic case
		preconditioner	 DIC;
		tolerance				 1e-06/1e-4;//loose for 1e-4 and tight for 1e-6
		relTol					 0/0.01; //0 for tight tolerance and 0.01 for loose. Start with loose then change to loose
	}
}
```



###### pFinal - if using **PISO or PIMPLE transient solvers**

+ final pressure correction - loose for intermediate corrector steps and put all the computational effort only in the last corrector step (pFinal)

```c++
solvers
{
	pFinal
	{
		$p; //macro pressure correction
    tolerance 1e-6; //tight tolerance
		relTol  0;
	}
}


PISO
{
  nCorrectors  2; //if loose during intermediate steps and tight on the final step, use at least "nCorrectors 2"
  nNonOrthogonalCorrectors  1; //depending on your mesh quality
}
```



###### U - asymmetric matrix (using asymmetric solvers) and inexpensive cost

```c++
solvers
{
	U
	{
		solver							PBiCGStab
    preconditioner			DILU;
    tolerance						1e-08;//inexpensive to compute asymmetric transported fields, so start right away with tight tolerance
    relTol							0;
	}
}
```









#### Pressure-velocity coupling in OpenFoam (solving partial differential equations)

As turbulent flows are governed by **Navier-Stokes equations**, which has the **nonlinear term (convection term)** that is the **main contributor** to the **turbulence** however causes numerical difficulties to solve. **Physically** the nonlinearity is due to **convective acceleration**, that is the acceleration associated with the change in velocity over position.



###### Ordinary differential equation 

an equation containing an unknown function of one real or complex variable x, its derivatives, and some given functions of x.



###### Partial differential equation

a differential equation that contains unknown multivariable functions and their partial derivatives. (This is in contrast to ordinary differential equations, which deal with functions of a single variable and their derivatives.)



###### The most widely used approaches for NSE

+ Pressure-based approach (predictor-corrector)

  + Two general methods: **Segregated method** and **Coupled method**

    **Segregated method** solves equations one after another or **one at a time**, memory efficient however bad convergence (in comparison to coupled solvers)

    **coupled method** solves continuity, momentum, and energy equation simultaneously, that is coupled together (currently in use) 

  + U field is obtained from the momentum equations (e.g. UEqn.H)

  + **SIMPLE, SIMPLEC, SIMPLER, PISO**

    **SIMPLE** and **SIMPLEC** are for **steady simulations**

    **PISO** and **PIMPLE** are for **unsteady simulations**

    note: PIMPLE is **more stable** than SIMPLE but higher computational cost

  

  **In comparison to PISO, making  corrections, SIMPLE makes 1 correction**

  ```c++
  SIMPLE
  {
    consistent yes; //SIMPLEC if yes (by default no)
  	nNonOrthogonalCorrectors 1;//at least 1 for non-orthogonal correction
  }
  ```

  ```c++
  PISO
  {
    momentumPredictor		yes; //help stabilizing the solution especially for high Re flows (strong convection)
    												 //you will need to define the linear solvers for the variables
  	nCorrectors  2;//for good accuracy 2 nCorrectors is recommended
  	nNonOrthogonalCorrectors 1;
  }
  ```

  ```c++
  PIMPLE
  {
    momentumPredictor		yes; //help stabilizing the solution especially for high Re flows (strong convection)
    												 //you will need to define the linear solvers for the variables
    nOuterCorrectors 1; //controls the loop outside, more outer corrections if using large time-steps, however high cost!
  	nCorrectors  2;//for good accuracy 2 nCorrectors is recommended
  	nNonOrthogonalCorrectors 1;
  }
  ```

  

  + default option in OpenFoam
  + intrinsically implicit

+ Density-based approach

#### VolScalarField::Internal

If `internal` method is used, `AverageField` class will fail to output `Mean` results.





