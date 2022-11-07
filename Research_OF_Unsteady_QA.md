# OpenFoam unsteady-simulation study

[toc]

**Note: The contour of Reynolds stress for DNS on slides for PWC meeting was wrong! Shuai has produced the correct version.**

### Introduction

For transient turbulent flows, **two options** of **transient solvers** are provided in OpenFoam: **PISO** vs **pimple**. With PISO, you must ensure **co < = 1** to ensure  numerical stability! Therefore, PISO is a more suitable option for **DNS** or **LES** simulations (capturing the detailed physics changing with time), however simulations take **so long** to finish due to such small time steps! If we want simulations run in faster pace, choose **pimple**. It combines PISO and simple methods, ensuring steady state to be reached at each time step. Note during each time step pimple takes longer than piso, however we can **set much larger time step** for compromise! To trigger pimple, do this

```c++
PIMPLE
{
    nCorrectors 1; //this does not result in "pimple" is activated (PISO has it too)
    nOuterCorrectors 2; //this outer loop correction is the signature for pimple!
}
```



```c++
 
PIMPLE
{
    nNonOrthogonalCorrectors 0; //for non-orthogonal mesh, use 1 - 3 depending on mesh quality
    nCorrectors          1; //1-3
    nOuterCorrectors    50;//larger number is chosen but "residualControl" must be provided to control
 
    residualControl
    {
        U
        {
                tolerance  1e-5;//this means once this specified tolerance is reached stop outer correcting the pressure!
                relTol      0;
        }
        p
        {
                tolerance  5e-4;
                relTol      0;
        }
     }
}
 
relaxationFactors
{
    fields
    {
        p      0.3;
        pFinal   1;//final step no relaxation factor is required
    }
    equations
    {
        "U|k|epsilon"     0.3;
        "(U|k|epsilon)Final"   1;
    }
}
```



#### When to choose pimple?

when you are dealing with very complex problems, bad initialized cases, weak boundary setup and wants to increase the co number.



#### Residuals in unsteady simulations

The following image shows residuals for several flow quantities running the **T3A** case using **pimple**. By comparing to **PISO**, it has shown that much less time was taken to reach convergence, since a much larger co number can be used with pimple. The image shows that as long as the residuals stop fluctuating and become steady, results **should not change** much. This does not mean we are getting accurate results!! We should always monitor the **quantities of interest** for unsteady simulations to have a better sense of our results. **Note residuals for unsteady flow need not to be very small as long as they become steady.** 

<img src="/Users/minghanchu/Documents/GitHub/C++learning/Research_OF_Unsteady_QA.assets/UQT3AResiduals.png" alt="UQT3A_pimple_Residuals" style="zoom:25%;" />



##### Very hard time April 10 2021

Issues:

Important observations

**Part 1**

1. **do not do: renumberMesh -overwrite** after a simulation blows up!

2. mesh quality, HPC(cedar or graham), number of cores, **boundary conditions**, **initial conditions**(I have changed the initial conditions later to the current version), type of flow (flat plate cannot do unsteady simulations with perturbations on) **all affect i**f this simulation will blow up!!! I am suffering from this result, struggling on it and still have not properly solved this problem. My time is very limited and I am very disappointed, frustrated, and extremely sad. I hope I can get over this soon.   

3. Note try to output by **writeInterval 500**, **do not use 250**.(Maybe the problem is due to time precision change 6 to 12)

4. keep the specified running time same after code blowing up, e.g. original asked 23:59 min and do not change it, just `sbatch`

5. Mesh quality is very important, keep it simple if possible



**April 15th 2021**

**Part 2**

So far the only mesh that might be used for research is the one created by Flavio. I am working on the refinement based on his original mesh, since the value of **yPlus** is around 2 within the bubble. Some conclusions drawn here:

1. Without changing the number of cells and only decreasing the distance between the first cell and the wall, obtained results however looked bad, i.e. in similar trend to my own 50w mesh.
2. Decreasing the distance between the first cell and the wall along with increasing the number of cells on airfoil surface, with increasing the **expansion ratio** from 0 to 1.1 (suggested by Menter 1.1 - 1.15), **cl and cd** exhibited a healthy trend until time > 0.3, curves went unstable and not tended to converge.
3. Decreasing the distance between the first cell and the wall along with increasing the number of cells on airfoil surface, withOUT increasing the **expansion ratio**, i.e. 0, **cl and cd** exhibited chaotic fluctuations after time > 0.1. 



**Plan**

1. keep increasing **expansion ratio** from 1.1 to 1.15, and see how result change

   **Observations:** more stable after increasing **ER** from 0.0 to 1.1.

2. increasing the number of cells on airfoil

3. increasing the number of cells in the wall normal direction

4. If still cannot find a proper mesh after numerous thorough big changes, then tune slightly, e.g. change from 0.0004 to 0.00035. 



**Done and conclusion**

1. Without changing the number of cells, I have reduced the distance at the first grid to **0.0003** and **0.0002**. Results looked good during transitional process however **cf curve** drops down below unphysical **zero** beyond 75% cord for **0.0003**, and beyond 80% cord for **0.0002**.

   **Note:** to resolve this issue I have made changes by reducing the grid size at the trailing edge, **cf curve** did go back up above zero but be still chaotic and unphysical. 

   **Next:**

   * increase 10 cells and 20 cells on either **airfoil** or in the **wall normal direction** and see if the **cf curve** can back up above zero around trailing edge.

     **observations**

     Increasing either 10 or 20 cells on airfoil will make the cf results unphysical (chaotic) above zero beyond c = 0.1

     Increasing either 20 cells (90 cells compared to 70 cells) in the wake region, cf gradually goes above zero with number of cells in the wake region, however the peak value decreases compared to the original case, i.e. y = 0.0004. **The width of plateau becomes wider which is unexpected**.

     Increasing 5 cells in the wall normal direction results in smaller peak value of cf and almost all part on the airfoil above zero. **The width of plateau becomes wider**.

   * Try a case of only y = 0.0003 without increasing number of cells and no changing in expansion ratio.

   * Try a case of y=0.0003 plus ER = 1.1

   * Try a case of 110 cells in the wake region

   * Try a case of wall normal cells = 110 

   * try a case of wall normal cells = 105 for y = 0.0004

     If y=0.0004 is kept untouched, only increasing the number of cells in the wall normal by a small number will not change the results much, e.g. 105  in the wall normal direction compared to 100. I have noticed that this model is fragile in terms of multiple-cups computation, meaning you may get stable results running on 1 cpu however results become unstable if multiple cpus are used. For example, with same boundary conditions and mesh I got stable, even though not very good, results with my 500k cells mesh, however on HPC results are chaotic. I do not think this is due to the unsteady solver. Only a small change, e.g. adding 10 cells or reducing the wall normal distance by a small amount (from 0.0004 to 0.0003), has resulted in big difference in results! This is very very annoying! Smaller y+ did not improve the results! Finer mesh only causes divergence! 

   * try a case of y = 0.00035 (maybe try this last)

2. After many tests, I have to use the **original mesh** to do my research. **yplus** is around 1.5 in the turbulent boundary layer

3. I have tested that **averaging field of wallShear remains unchanged beyond time > 0.8 with averaging from t = 0.5**, it turned out that the **averaged wallShear** at **t=0.8** is same as that at **t = 1.17**. 



**April 29th 2021**

Using the mesh created by Flavio, 

1. All cases start collecting data from t = 0.6 to 1.4/1.5.
2. 1c gave most intensive perturbation, higher than the baseline curve, compared to the baseline
3. 3c gave the least intensive perturbation compared to the baseline
4. 2c gave the medium degree of perturbation, but still higher than the baseline
5. 1c1p gave smaller perturbation (however larger than 3c)
6. 2c1p gave...



**May 4th 2021**

Turbulence intensity testing:

Currently the one that works for both baseline and perturbed cases is at Tu = 0.027%, the smallest degree of turbulence intensity that the two-equation transition model can stand.

+ I have tried a larger value of Tu = 0.5%, k increased by 2 orders of magnitude, Rethetat decreased to 880, omega increased by more than 10 times of magnitude (better baseline results obtained compared to LES/DNS)
+ However with Tu = 0.5% 1c case crashed at time = 0.68
+ then reduced value of omega from 7 to 1, still crashed at time = 0.68, and results did not seem change much, meaning value of omega has little importance affecting code crashing and final results
+ I am currently testing Tu = 0.1% having smaller value of k and Rethetat, 
+ from the above piece of information I maybe decrease k and hold Rethetat constant, or vice versa







Note all above observations are based on the **"backwardFacingstep" and "UQT3A" and "UQSD7003"**

After implementing UQ into unsteady solver of `pimpleFoam`, running **UQSD7003** 

