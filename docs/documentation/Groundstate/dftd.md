# Long-range dispersion corrections

`TODOS`     

 * do the TSSURF and TSSCS schemes work/do anything?
 * are the aperiodic versions of TSSCS and MBD still a thing? 
 * What elements are supported for D3? 
 * General recommendations for types of systems -- are these ok? Add others?   


## Background

### Why and when are long range-dispersion methods needed?
van der Waals (vdW) interactions are ubiquitous in nature but aren't accounted for by standard local and semilocal density functional approximations (i.e. LDA or GGAs). In particular, the long-range attractive part of the vdW interaction between system components (London dispersion interaction) is not captured. Including such dispersion effects is essential for accurately describing certain systems, especially for systems that are not bound ionically or covalently, e.g. gas molecules. 

The use of a dispersion method is strongly recommended when intermolecular interactions are expected to play a key role such as in molecular crystals, stacked 2D materials, and generally any weakly-bound system.


### Types of correction schemes
There are several families of dispersion methods. A good summary of these can be found in the [2016 review by Grimme et al.](https://doi.org/10.1021/acs.chemrev.5b00533) We will summarise these categories here and, for those implemented in CASTEP, link to the relevant keywords.


**Semiempirical/semiclassical treatments**
Here the dispersion energy is calculated atom-wise (typically pair-wise) and is added to the DFT energy of the underlying functional. Note that a specific damping parameter (and function) is usually used to combine a given underlying functional (e.g. PBE) with a dispersion correction (e.g. D3). This means, for example, that the D3 correction to the PBE functional may be different to the D3 correction to HSE.  

Methods include:

* The Tkatchenko−Scheffler scheme ([TS](#TS))    
* The TS-based many-body dispersion scheme ([MBD](#MBD))    
* Grimme's [D2](#G06) and [D3](#D3) methods    
* The exchange-dipole moment model ([XDM](#XDM))   


**Nonlocal density-based treatments (not implemented in CASTEP)**

Here non-local functionals of the electronic density are constructed. Methods include the van der Waals functionals (e.g. [`vdW-DF2`](https://doi.org/10.1103/PhysRevB.82.081101) and [`vdW-DF-cx`](https://doi.org/10.1103/PhysRevB.89.035412)) and Vydrov and Van Voorhis functionals (e.g. [`VV10`](https://doi.org/10.1063/1.3521275)). 

**Effective one-electron potentials (not implemented in CASTEP)**
Finally, the many-body correlated motion of electrons can be, to some extent, captured empiricially by using effective one-electron potentials. Examples of these include: semi-local functionals such as the Minnesota functionals (e.g. [`M06`](https://doi.org/10.1007/s00214-007-0310-x)) and external potentials such as the dispersion-corrected atom-centered potentials (see e.g. [van Santen 2015](https://doi.org/10.1021/acs.jpca.5b02809)).

## General recommendations

`TODO -- check these`

Every application will have its own set of important features and it is not possible to say with certainty which density functional approximation will be the most appropriate. With that said, here are some suggestions for different types of materials:

  * Molecular crystals: D3/D3-BJ/XDM/MBD in combination with GGA or hybrid functionals ([Dolgonos et al. 2019](https://doi.org/10.1039/C9CP04488D)).
  * Metals: vdW-DFs developed with solid state materials in mind (see e.g. [Klimeš et al. 2011](http://dx.doi.org/10.1103/PhysRevB.83.195131)). `TODO: what about methods available in CASTEP?`
  * Interactions between molecules and surfaces of metals: vdW-DFs or dispersion methods that include screening, e.g. TSSCS
  * MOFs: dispersion corrected (e.g. TS, D3, etc.) GGAs
  * Layered vdW materials: Beyond pairwise appraches, e.g. D3$^*$ or MBD. 
  * `TODO: others?`

$^*$ See note below for how to include the three-body terms in the CASTEP D3 correction. 

Again, please do your own testing and consult relevant reviews and benchmarking papers. 


For the TS, MBD and XDM methods you may get a warning if your unit cell is small (where the lattice constants are comparable to the vdW radii). This has been fixed in academic CASTEP v24.1 for TS and MBD, and in v25 for XDM. If you are using an older version of CASTEP, then you should try to use larger unit cells until your dispersion correction converges.


Not all `XC_FUNCTIONAL` values are supported for all schemes - if in doubt use PBE. The primary 'XC_FUNCTIONAL' for XDM is B86PBE.    

Each scheme has default parameters defined for a subset of elements (see [table](#table) below). For other elements you need to define custom parameters using the `SEDC_CUSTOM_PARAMS` keyword in the .cell file. 

## Using dispersion corrections in CASTEP

### .param file keywords

In the `.param` file, set:

**`SEDC_APPLY`**` : true` turns on dispersion correction.    
**`SEDC_SCHEME`**: The semi-empirical dispersion/van der Waals correction scheme to use. Default is `NONE`, other possible values listed in the table below.


| SEDC_SCHEME<a name="table"></a>  | Available from CASTEP version                          | Compatible XC functionals                                | Elements supported                    | Stresses | Phonons<sup>*</sup>   |
|---------------------------|--------------------------------------------------------|----------------------------------------------------------|---------------------------------------|----------|-----------|
| [TS](#TS)                 | Predates 2012                                          | PBE, PBE0, PBE1PBE, BLYP, B3LYP, AM0, RPB, PBESOL, PW91  | Up to Z=54 with first row Lanthanides | Analytic | DFPT & FD |
| [TSSURF](#TSSURF)         | Not sure if working                                    |                                                          |                                       |          |           |
| [TSSCS](#TSSCS)           | Not sure if working                                    |                                                          | Up to Z=54 with first row Lanthanides | Analytic |           |
| [MBD](#MBD) (Alias: MBD*) | 2015? Rewritten (and fixed) for CASTEP 18              | PBE, PBE0, PBE1PBE, HSE03, HSE06, RSCAN                  | Up to Z=54 with first row Lanthanides | Numeric  | FD        |
| [G06](#G06) (= D2)        | Predates 2012                                          | PBE, BLYP,  BP86, B3LYP, TPSS                            | Up to Z= 54                           | Analytic | DFPT & FD |
| [D3](#D3)                 | CASTEP 20 ([Compilation instructions](#D3compilation)) | PBE, PBE0, HF                                            |                                       | Analytic | FD        |
| [D3-BJ](#D3-BJ)           | CASTEP 20 ([Compilation instructions](#D3compilation)) | PBE, PBE0, HF                                            |                                       | Analytic | FD        |
| [OBS](#OBS)               | Predates 2012                                          | LDA, PW91                                                | Up to Z=57                            | Analytic | DFPT & FD |
| [JCHS](#JCHS)             | Predates 2012                                          | PBE, BLYP, B3LYP, TPSS                                   | H, C, N, O, F, Cl, Br                 | Analytic | DFPT & FD |
| [XDM](#XDM)               | CASTEP 20                                              | B86PBE, PBE, BLYP, PBESOL, PW91, RPBE, WC, RSCAN         | Up to Z=102                           | Analytic | FD        |
<sup>*</sup> DFPT: Density functional perturbation theory; FD: finite displacement.    


All of the above methods support analytic forces. 

### .cell file keywords

In the `.cell` file you can *optionally* set custom parameters for each species:

* `C6` (eV Å$^6$): Available for TS, MBD*, and Grimme schemes.
* `R0` (Å): Available for TS, MBD*, and Grimme schemes.
* `alpha` (Å$^3$): Available for TS, MBD*, and OBS schemes.
* `I` (eV): Available for the OBS scheme.
* `Rvdw` (Å): Available for the OBS scheme.

You set them like this:

```
%BLOCK SEDC_CUSTOM_PARAMS
      ! example values for test purposes only - don't use these... !
      He      C6:1.00      R0:2.00
      Ne      C6:10.00     R0:4.00
%ENDBLOCK SEDC_CUSTOM_PARAMS
```


## Method details and customisation


#### Tkatchenko−Scheffler schemes
A density-dependent, atom pairwise dispersion-correction scheme.

**`SEDC_SCHEME : TS`**<a name="TS"></a>     
[Phys. Rev. Lett.,102, 073005 (2009)](https://doi.org/10.1103/PhysRevLett.102.073005)

Restrictions: 

* Can only be used with the following XC functionals: PBE, PBE0, PBE1PBE, BLYP, B3LYP, AM0, RPB, PBESOL, PW91    
* Elements up to Z=54 (Xe) together with first row Lanthanides are supported.

**`SEDC_SCHEME : TSSCS`** # TS with self-consistent screening<a name="TSSCS"></a>    
[Phys. Rev. Lett., 108, 236402 (2012)](https://doi.org/10.1103/PhysRevLett.108.236402)    


**`SEDC_SCHEME : TSSURF`**<a name="TSSURF"></a>    
 [Phys. Rev. Lett., 108, 146103 (2012)](https://doi.org/10.1103/PhysRevLett.108.146103)


??? keyword "Customisation keywords for all TS schemes"

    **.param file**   
    
    `SEDC_SR_TS`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable SR value for the damping function in the TS
            semi-empirical dispersion/van der Waals correction scheme.    
      Modifiable: restart and on the fly     
      Default is determined by `XC_FUNCTIONAL`.

    `SEDC_D_TS`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable d value for the damping function in the TS
            semi-empirical dispersion/van der Waals correction scheme.    
      Modifiable: restart and on the fly     
      Default is determined by `XC_FUNCTIONAL`.

      **.cell file**

      `SEDC_CUSTOM_PARAMS`     
        Type: Block    
        Level: Basic   
        Description: Customized parameters for semi-empirical dispersion corrections. Used to calculate van der Waals forces. Default value for each species is determined by the chosen semiempirical correction scheme. For the TS methods, `C6` (eV Å$^6$), `R0` (Å) and `alpha` (Å$^3$) can be customised.








#### Many-body dispersion <a name="MBD"></a>

Beyond what can be included in simple pairwise dispersion approaches, MBD introduces: (1) many-body energy (Axilrod-Teller and higher-order) and (2) long-range Coulomb response (screening) that serves to modify the polarizabilities of interacting species.


**`SEDC_SCHEME : MBD`** # (Alias: MBD*)     
[Phys. Rev. Lett., 108, 236402 (2012)](https://doi.org/10.1103/PhysRevLett.108.236402)    
[J. Chem. Phys. 140, 18A508 (2014)](https://aip.scitation.org/doi/10.1063/1.4865104)    

Note that the two aliases MBD and MBD* both refer to the revised version of MBD which employs range-separation (rs) of the self-consistent screening (SCS) of polarizabilities and the calculation of the long-range correlation energy, i.e. the MBD@rs-scs method.

`TODO`: in older versions of CASTEP was this the case? From what version can be rely on that info? 

Restrictions: 

* Can only be used with the following XC functionals: PBE, PBE0, PBE1PBE, HSE03, HSE06, RSCAN    
* Elements up to Z=54 (Xe) together with first row Lanthanides are supported.
* Note that in the current implementation, only numeric stresses are available.


??? keyword "Customisation keywords for MBD scheme"

    **.param file**   
    
    `SEDC_SR_TS`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable SR value for the damping function in the TS and MBD correction schemes.    
      Modifiable: restart and on the fly     
      Default is determined by `XC_FUNCTIONAL`.

    `SEDC_D_TS`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable d value for the damping functionin the TS and MBD correction schemes.      
      Modifiable: restart and on the fly     
      Default is determined by `XC_FUNCTIONAL`.

      **.cell file**

      `SEDC_CUSTOM_PARAMS`     
        Type: Block    
        Level: Basic   
        Description: Customized parameters for semi-empirical dispersion corrections. Used to calculate van der Waals forces. Default value for each species is determined by the chosen semiempirical correction scheme. For the MBD method, `C6` (eV Å$^6$), `R0` (Å) and `alpha` (Å$^3$) can be customised.





#### Grimme D corrections
**D2**

Note that default D2 parameters are available only for the PBE, BLYP, BP86, B3LYP, TPSS functionals. You can also customise the correction parameters using the following keywords:

**`SEDC_SCHEME : G06`** # This is CASTEP's name for the Grimme D2 correction<a name="G06"></a>     
[J. Comput. Chem. 27, 1787, (2006)](https://doi.org/10.1002/jcc.20495)

??? keyword "Customisation keywords G06"

      **.param file**   
    
      `SEDC_S6_G06`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable S6 value for the damping function in the G06
            semi-empirical dispersion/van der Waals correction scheme.    
      Modifiable: restart and on the fly     
      Default is determined by `XC_FUNCTIONAL`.

      `SEDC_D_G06`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable d value for the damping function in the G06
            semi-empirical dispersion/van der Waals correction scheme.    
      Modifiable: restart and on the fly     
      Default is determined by `XC_FUNCTIONAL`.

      **.cell file**

      `SEDC_CUSTOM_PARAMS`     
        Type: Block    
        Level: Basic   
        Description: Customized parameters for semi-empirical dispersion corrections. Used to calculate van der Waals forces. Default value for each species is determined by the chosen semiempirical correction scheme. For Grimme's D2 method, `C6` (eV Å$^6$) and `R0` (Å) can be customised. 


**D3**

Grimme's DFT-D3 method. The D3 scheme with Becke-Johnson damping (D3-BJ) is [generally more accurate](https://pubs.acs.org/doi/full/10.1021/acs.jpclett.6b00780) than the 'zero-damping' method (D3).


Restrictions:    

 * Default D3 parameters are available for the PBE, PBE0 and HF functionals.
 * `TODO: what elements are supported?`
 * By default the **three-body term is not included**. To include it, set `d3_threebody: True` within a `devel_code` block in the .param file. Turn on `IPRINT > 1` to get more information on what has been included in the D3 correction.
 * In the current implementation, users cannot supply custom parameters for this correction scheme.


**`SEDC_SCHEME : D3`**<a name="D3"></a>     
[J. Chem. Phys. 132, 154104 (2010)](https://doi.org/10.1063/1.3382344)    

**`SEDC_SCHEME : D3-BJ`**<a name="D3-BJ"></a>    
[J. Comput. Chem. 32, 1456 (2011)](https://doi.org/10.1002/jcc.21759)

This is the Grimme D3 scheme with Becke-Johnson damping.

!!! note 
      When running a D3-BJ calculation with `IPRINT > 1`, you might see `Dispersion version: D4` in the .castep output file. Confusingly, this does **not** mean the Grimme D4 method has been used, it's just an interal CASTEP version label for this correction scheme.  


??? warning "Compilation flags: for the D3 correction, CASTEP must be compiled with `GRIMMED3 := compile`<a name="D3compilation"></a>"
      
      Note that for the D3 correction to work, CASTEP must be compiled with the following flag:
      ```
      # Grimme D3 library support. Options are none or compile
      GRIMMED3 := compile
      ```
      in the `Makefile`

      For CASTEP 20, the D3 library is included in the distribution and the setting the flag above should be sufficient. 


#### Exchange-dipole moment (XDM) method <a name="XDM"></a>

A unified density-functional treatment of dynamical, nondynamical, and dispersion correlations.


Restrictions: 

* Default XDM parameters are available for the following XC functionals: B86PBE, PBE, BLYP, PBESOL, PW91, RPBE, WC, RSCAN

**`SEDC_SCHEME : XDM`**    
[J. Chem. Phys. 127, 124108 (2007)](https://doi.org/10.1063/1.2768530)

For functionals that are not in this list, you need to specify the following additional parameters: 

??? keyword "Customisation keywords"

      **.param file**   

      `SEDC_A1_XDM`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable A1 value for the damping function in the XDM
            semi-empirical dispersion/van der Waals correction scheme.       
      Modifiable: restart and on the fly        
      Default is determined by `XC_FUNCTIONAL`.

      `SEDC_A2_XDM`    
      Type: Physical (float)    
      Level: Expert   
      Description:
            Customisable A2 value for the damping function in the XDM
            semi-empirical dispersion/van der Waals correction scheme.        
      Modifiable: restart and on the fly          
      Default is determined by `XC_FUNCTIONAL`.

      `SEDC_SC_XDM`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable SC value for the damping function in the XDM
            semi-empirical dispersion/van der Waals correction scheme.       
      Modifiable: restart and on the fly        
      Default is determined by `XC_FUNCTIONAL`.

      `SEDC_C9_XDM`    
      Type: Logical    
      Level: Basic   
      Description:
            Specifies whether three-body dispersion coefficients are 
            to be computed in the XDM semi-empirical dispersion/ 
            van der Waals correction scheme.       
      Modifiable: restart and on the fly        
      Allowed values: `TRUE` or `FALSE`    
      Default is `FALSE`


Note that if you wish to experiment with these values, for example to test a new functional, then careful further testing is necessary.  

#### Other Schemes
**`SEDC_SCHEME : OBS`** # (Ortmann, Bechstedt and Schmidt)<a name="OBS"></a>    

Semiempirical van der Waals correction to the density functional description of solids and molecular structures

[Phys. Rev. B 73, 205101, (2006)](https://doi.org/10.1103/PhysRevB.73.205101)
??? keyword "Customisation keywords"

    **.param file**   

    "`SEDC_LAMBDA_OBS`"   
        Type: Real (float)   
        Level: Expert   
        Description:
                Customisable lambda value for damping function in the the OBS
                semi-empirical dispersion/van der Waals correction scheme.     
          Modifiable: restart and on the fly        
          Default is determined by `XC_FUNCTIONAL`.

    "`SEDC_N_OBS`"   
        Type: Real (float)   
        Level: Expert   
        Description:
              Customisable n value for the damping function in the OBS
              semi-empirical dispersion/van der Waals correction scheme.    
        Modifiable: restart and on the fly        
        Default is determined by `XC_FUNCTIONAL`.

**`SEDC_SCHEME : JCHS`** # (Jurečka, Černý, Hobza and Salahub)<a name="JCHS"></a>   

[J. Comput. Chem. 28, 555, (2007)](https://doi.org/10.1002/jcc.20570)

??? keyword "Customisation keywords"

    **.param file**   

    `SEDC_SR_JCHS`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable SR value for the damping function in the JCHS
            semi-empirical dispersion/van der Waals correction scheme.       
      Modifiable: restart and on the fly        
      Default is determined by `XC_FUNCTIONAL`.

    `SEDC_S6_JCHS`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable S6 value for the damping function in the JCHS
            semi-empirical dispersion/van der Waals correction scheme.       
      Modifiable: restart and on the fly        
      Default is determined by `XC_FUNCTIONAL`.

    `SEDC_D_JCHS`    
      Type: Real (float)    
      Level: Expert   
      Description:
            Customisable d value for the damping function in the JCHS
            semi-empirical dispersion/van der Waals correction scheme.       
      Modifiable: restart and on the fly        
      Default is determined by `XC_FUNCTIONAL`.







