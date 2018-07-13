# Imaging algorithm of CLP-SECM data
---
## 1. Overview
This code package is dedicated to image reconstruction from scanning chemical microscopic imaging (SECM) data measrued by continuous line probe (CLP). A CLP-SECM scan is setup by putting the sample on a rotational stage and locate the line probe at one side of sample. A single scan line is generated as follows: The microscope firstly rotate its stage by preassigned angle, then followed by stepwise moving CLP to the other end of sample with constant stepsize. At each steps, CLP measures the current generated by electrochemical reaction beween sample and probe end. A line scan is the collection of all measuresments at all steps with sample rotation fixed at some given angle. Combining signle line scans under different sample rotation angles gives the complete CLP-SECM scan.

In our experiments, the interested sample---chemical reactive species attached on substrate---is highly structued. More specifically, the shape reactive species are known, are much smaller in comparison with the sample size, and also are sparsely populated on the substrate. A sample **Y** can therefore being modeled as 2D-convolution between the reactive species **D** and a sparse activation map  **X<sub>0</sub>**, denote as **Y = D\*X<sub>0</sub>**. We adopt compressed sensing methodology, to reconstruct the SECM sample image by finding the exact locations map  **X<sub>0</sub>**. Define CLP line scan **L** takes the sample **Y** and CLP properties **p**---scanning angles, point-spread-funciton of CLP, etc., and generates measured current lines **R** where 

<p align="center"><strong>                         R = L[ D*X<sub>0</sub>, p].                                     </strong></p>

The reconstruction algorithm gathers CLP-SECM scan lines **R** to perform image reconstruction using numerical optimization procedure. The algorithm formulate the task of finding the location of the reactive species as a variation of LASSO problem. Namely, the algorithm solves the following problem

<p align="center"><strong>    min<sub>{X>0,p}</sub>   C||X||<sub>1</sub> + ||L[D*X-R, p]||<sup>2</sup>             </strong></p>

where **C** is some positive constant, **||X||<sub>1</sub>** denotes total magnitude of **X**, and **||R||<sup>2</sup>** represents energy of lines **R**. We claim that the sample image reconstruction is successful, if the location (non-zero entries) of **X** is identical to **X<sub>0</sub>**. 

This code package uses *inertial proximal alternating linearized minimization* (IPALM) to solve aforementioned optimization problem.


## 2. Data Management
Create a folder `clpsecm_data` under the same path to folder `clpsecm_algorithm`: 
```
./path/clpsecm_data 
./path/clpsecm_algorithm
```
Put the data in excel file format in the folder with the naming convention:
```
clpsecm_(MMDDYY)_S(sample_number)_L(number_of_lines)_(data_number).xlsx
```
For example, a CLP-SECM file is generated in date July 04, 2076, with scans the ten dots sample with sample number 3-2. The scan choose 10 lines with different angles, and is the second experiment in that same day under same setting, should has the file name defined as
```
clpsecm_070476_S032_L10_02.xlsx
```
In the file, each of the line scans is stored in separated sheets started from sheet 2, where column B,C records the scanning distance and reactive currents, respectively.

## 3. Overview of Code Package
1. The SECM image objects, in the path `../_algorithm/_classes/data/`, contains the following class objects:
      1. `DataSpec`    - The specification of target data. It reads the datafile and produce a `ScanLine` object.
      2. `SecmCoord`   - Coordinate system of SECM.
      3. `ScanLines`   - The lines generated by CLP.
      4. `SecmImage`   - The SECM image under the coordinate system.
      5. `Sparsemap`   - The Binary SECM image that has a few sparsely populated non-zero entries, used to represent **X<sub>0</sub>**.
      6. `DictProfile` - The SECM image of a single dictionary profile with center locating at (0,0), used to represent **D**.
      7. `ProbeParam`  - Record single CLP parameter, there are four types of parameter controls behavior of CLP:
            * Scan angle.
            * Line shifts.
            * Pointwise measurement intensity.
            * Point-spread-function of line scan.
      8. `ProbeParams` - Record all four CLP parameters.   
      
    The SECM image objects has inheritance hierarchy as:
```
handle > DataSpec
       > SecmCoords > ScanLines 
                     > SecmImage > SparseMap
                                 > DictProfile
       > ProbeParam
       > ProbeParams
```   

2. The algorithm objects, in the path `../_algorithm/_classes/alg`, contains the following class objects:
      1. `SmcProblem` -  An abstract of problem which minimize objective contains smooth coupling term across variables.
      2. `CalibLasso` -  A minimization problem with lasso-type objective that calibrate linear measurements.
      3. `Solver`     -  A generica iterative based algorithmic environment.
      4. `Ipalm`      -  The IPALM solver, with which we solve problem of class `SmcProblem`
      
    The algorithm objects has inheritance hierarchy as:
```
handle > SmcProblem > CalibLasso
       > Solver     > Ipalm
```

## 4. Use of code
### 1. Basic usage

