# Imaging algorithm of CLP-SECM data
---
## 1. Overview
This code package is dedicated to image reconstruction from scanning chemical microscopic imaging (SECM) data measured by continuous line probe (CLP). A CLP-SECM scan is setup by putting the sample on a rotational stage and locate the line probe at one side of sample. A single scan line is generated as follows: The microscope firstly rotate its stage by preassigned angle, then followed by stepwise moving CLP to the other end of sample with constant stepsize. At each steps, CLP measures the current generated by electrochemical reaction beween sample and probe end. A line scan is the collection of all measuresments at all steps with sample rotation fixed at some given angle. Combining signle line scans under different sample rotation angles gives the complete CLP-SECM scan.

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
For example, a CLP-SECM file is generated in date July 04, 2076, which scans the ten dots sample of sample number 3-2. The scan consists of 10 lines with different angles, and is the second experiment in that same day under same setting, should has the file name defined as
```
clpsecm_070476_S032_L10_02.xlsx
```
In the file, each of the line scans is stored in separated sheets starting from sheet 2, where column B,C records the scanning distance and reactive currents, respectively.

## 3. Overview of Code Package
#### 1. The SECM image objects
In the path `../_algorithm/_classes/data/`, contains the following class objects:
1. `DataSpec`    - The specification of target data. It reads the datafile and produce a `ScanLines` object.
2. `SecmCoord`   - Coordinate system of SECM.
3. `ScanLines`   - The lines generated by CLP.
4. `SecmImage`   - The SECM image under the coordinate system.
5. `Sparsemap`   - The binary SECM image that has a few sparsely populated non-zero entries, used to represent **X<sub>0</sub>**.
6. `DictProfile` - The SECM image of a single dictionary profile with center locating at (0,0), used to represent **D**.
7. `ProbeParam`  - Record single CLP parameter, there are four types of parameter controls behavior of CLP:
      * Scanning angle.
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

#### 2. The algorithm objects
In the path `../_algorithm/_classes/alg`, contains the following class objects:
1. `SmcProblem` -  An abstract of problem which minimize objective contains smooth coupling term across variables.
2. `CalibLasso` -  A minimization problem with lasso-type objective that calibrate linear measurements.
3. `Solver`     -  A generic iterative based algorithmic environment.
4. `Ipalm`      -  The IPALM solver, with which we solve problem of class `SmcProblem`.
      
The algorithm objects has inheritance hierarchy as:
```
handle > SmcProblem > CalibLasso
       > Solver     > Ipalm
```

#### 3. Configuration of the continuous line probe
The m-file `clpconfig.m` records the parametric setting of CLP, which setup the scanning angles, shifts, intentisy and point-spread-function for line probe. Whenever a `ScanLines` object is created without specifically speccified parameter, this file is being read and produces the parametric setting accordingly.

## 4. Use of code
#### 1. Basic usage
We provides three basic examples for beginner to get familiar with the package:
* `Example1` - Read the data with given file name, generates `ScanLines` object and its back projected `SecmImage` object.
* `Example2` - Generate a synthetic SECM sample `SecmImage` object and produces a simluated line scans `ScanLines` object with known  parametric setting with object `ProbeParams`. Define a problem using object `CalibLasso`, and solve it with `Ipalm` algorithmic methods. The location of **X<sub>0</sub>** is recovered after sufficient number of iterations.
* `Example3` - Similar to `Example2`, but this time we setup problem `CalibLasso` with wrong guess of parameters. We show the algorithm still successfully find out the correct locations.

#### 2. Modify the configuration of line probe
In the file `clpconfig.m`, there are four type of parameters corresponding to the properties of line probe:
1. `angles`: Rotate the stage with specified angle and perform line projection.
2. `shifts`: Shift the currents for each lines by designed distance.
3. `intensity`: Assign the intensity for currents for every lines and every single measurements.
4. `psf`: the point spread function of currents, each lines convolute with the input kernel.
Users are free to modify the configuration, whenever the `ScanLines` object is created, if not specified, this file will be loaded and the CLP will equipped with these parametric setting when operation line projection and back projection.

#### 3. Create your own experiments
In `Example2` and `Example3`, we call the object `IpalmSecmSimu` as problem solvern, which is a children of `Ipalm` solver that is specially taylored to demonstrate result for simulated problems. For other experimental setup, users are encouraged to create other solver objects and deom the procedure of solver according to their own needs.

#### 4. Augment new properties for the CLP
Currently, the full line scan of CLP follows these operation sequentially. Whenever the following code is executed
```
lines = SecmImage.line_project(param);
```
then it generates the complete line scans under following procedure:
1. Rotate and projection: Rotate the stage with specified `angles` values and perform line projections.
2. Shift all lines: Shift lines projected in different angles by values of `shifts`.
3. Pointwise intensity: Each measurements is reweighed by 2D-function of `intensity`.
4. 1D-Convolution with point-spread-function: Each lines is convololved with 1D-function of `psf`.
Similarly, the back projection from the lines to image, executes
```
image = ScanLines.back_project();
```
then above scanning procedure operates in reverse order. In fact, the back projection is the *adjoint operator* of line projection. User can change the function both function `SecmImage.line_project()` and `ScanLines.back_project()` to modify or even augment new CLP properties.

---
<p align="right"> Compatibility: MATLAB R2018a or above. </p>
<p align="right"> Last Update: 2018-07-14 by Henry Kuo </p>

