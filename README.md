
# Topology optimization with ToOptix


<p align="center">
  <img src="https://github.com/DMST1990/ToOptixUpdate/blob/master/Images/StaticLoadCaseTwoRectangle.png" width="100%">
</p>


<p align="center">
  <img src="https://github.com/DMST1990/ToOptixUpdate/blob/master/Images/HeatLoadCaseTwoRectangle.png" width="100%">
</p>


<p align="center">
  <img src="https://github.com/DMST1990/ToOptixUpdate/blob/master/Images/QuadraticPlateLoadCase.png" width="100%">
</p>

<p align="center">
  <img src="https://github.com/DMST1990/ToOptixUpdate/blob/master/Images/NoDesignSpace.png" width="100%">
</p>




## Current version
- Only 3D-FEM support
- Heat Transfer
- Static load case
- Material will change only in young module and conductivity
- Filter selects only the element around the filter object

## Installation
- Install python 3.xx
- Download ToOptix

### General information
ToOptix core Functions: Python libary with the core functions of tooptix
[ToOptix Core](https://github.com/DMST1990/ToOptixCore/)

- Start this program in a user directory so Blender should be for example on the desktop 
- If you want to start Tooptix in "C:Programms\Blender Foundation ..." you need administrator rights (not reccomended)
- So i would suggest you should take a copy of blender and then use it on the desktop or some other user access folder
- (Optional) create a environment variable for Calculix (ccx)
- Test at first the two example files TwoRectanglesStruc.inp and TwoRectanglesTherm.inp


### Blender Installation

Location of the Blender Addon
[Blender Addon](https://github.com/DMST1990/ToOptixBlenderAddon)


- Only supported for Blender 2.80
- Copy the folder ToOptix_BlenderAddon
- Copy your ToOptix Folder and paste it into ...\Blender Foundation\Blender\2.80\scripts\addons
- Start Blender and activate the addon (type = mesh)
<p align="center">
  <img src="https://github.com/DMST1990/ToOptixUpdate/blob/master/Images/Blender_UI.PNG" width="50%">
</p>

### Python 
[ToOptix Core](https://github.com/DMST1990/ToOptixCore/)
- Check if import statement of run_optimization.py is: 
from TopologyOptimizer.OptimizationController import OptimizationController 
- Open the folder with pycharm and just start your optimization
- Use the Folder ToOptix_Python

Example no design space with file:

```python,example

from run_optimization import run_optimization

cpus = 4

# Optimization type --> seperated (combined is only in beta )

opti_type = "seperated"
# no design space is used until redefinition
sol_type = ["no_design_space", "static"]
files = ["no_design_space.inp", "testinp\Cylinder_Mesh.inp"]
max_iteration = 20
vol_frac = 0.3
penal = 3.0
matSets = 10
weight_factors = [3.0]
workDir = "work"
solverPath = "ccx"
run_optimization(penal,  matSets, opti_type, sol_type,
                                      weight_factors, max_iteration, vol_frac,
                                      files, workDir, solverPath, cpus)
```
Example no design space with element set and several iterations:

- The element set created by FreeCAD will be named as follow
'SolidMaterialSolid'  'SolidMaterial001Solid'  'SolidMaterial002Solid'  
You can specify the non design space by another material definition. So that SolidMaterialSolid is free and SolidMaterial001Solid is a non design space.
- You can specify your own non design space by creating a element set in '.inp' file and use the name of the element set as the non design space definition

```python,example

from run_optimization import run_optimization

# Optimization type --> seperated (combined is only in beta )
cpus = 6
opti_type = "seperated"
sol_type = ["static"]
files = ["PlateWithNoDesignSpaceFine.inp"]

for vol_frac in [0.4, 0.6]:
    for penal in [3.0]:
        max_iteration = 100
        matSets = 20
        weight_factors = [1.0]
        workDir = "work"
        solverPath = "ccx"
        no_design_set = 'SolidMaterial001Solid'
        run_optimization(penal,  matSets, opti_type, sol_type,
                                              weight_factors, max_iteration, vol_frac,
                                              files, workDir, solverPath, cpus, no_design_set)


``` 

## For using the python FreeCAD Macro 
- This Macro is tested on FreeCAD 0.17. Later version might not work.
- You need to define a python3 path 'python3_path' in the file 'FreeCADMacro.py' because FreeCAD uses python2 as default
- You need to define a install path 'installation_path' in the file 'FreeCADMacro.py' that FreeCAD knows where ToOptix is located.
- During the macro FreeCAD changes the directory to the ToOptix folder and creates a 'config.json' file with the 'model.inp' path and the 'ccx.exe' path . 
- Create a FEM Model (which should work) for a static load case. Then run the Macro 'FreeCADMacro.py' in FreeCAD
- FreeCAD Macro, you can specifiy a non design material as well, just like before.
- Modification should be done in 'run_optimization_freeCAD.py'. This will be changed in the next version.

<p align="center">
  <img src="https://github.com/DMST1990/ToOptixUpdate/blob/master/Images/InstructionFreeCADMacro.png" width="100%">
</p>




```python, 

from run_optimization import run_optimization
import json
import os


json_path = 'config.json'
if __name__ == "__main__":
    # Optimization type --> seperated (combined is not implemented )
    cpus = 6
    opti_type = "seperated"
    sol_type = ["static"]
    with open(json_path, 'r') as file:
        data = json.load(file)
    files = [data['inp_path']]
    workDir = 'work'
    solverPath =  "\"" + str(data['ccx_path']) +  "\""
    inp_path = data['inp_path']
    files = [inp_path]
    for vol_frac in [0.4]:
        for penal in [3.0]:
            max_iteration = 100
            matSets = 20
            weight_factors = [1.0]
            no_design_set = 'SolidMaterial001Solid'
            no_design_set = None
            run_optimization(penal,  matSets, opti_type, sol_type,
                                                  weight_factors, max_iteration, vol_frac,
                                                  files, workDir, solverPath, cpus, no_design_set)
```


## Using rendering module mayavi
- Only supported if mayavi and trimesh is insatlled
- This module creates rendered pictures of the result into the working folder.
- Install mayavi package (with vtk ...)
- Install trimesh package (for converting the stl file)

- goto the file 'TopologyOptimizer\OptimizationController.py'
- At the beginning of the file set the variable 'use_trimesh_may_avi=True'
- You can specify a 'interactive_visualization_after_iteration'. After these iteration interval a scene will be poped up and rendered.

<p align="center">
  <img src="https://github.com/DMST1990/ToOptixUpdate/blob/master/Images/RenderedMAYAVI.jpg" width="50%">
</p>


```python, 

use_trimesh_may_avi = True
if use_trimesh_may_avi:
    from mayavi import mlab
    import trimesh
    interactive_visualization_after_iteration = 10
```
## Output
- STL File in a specific folder for every optimizaiton step
- Rendered Pictures of the result




