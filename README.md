
# g(raph) - g(cbh)

gg is an open-source code for building graph-based grand canonical basin hopping calculator

[![Documentation Status](https://readthedocs.org/projects/graph-gcbh/badge/?version=latest)](https://graph-gcbh.readthedocs.io/en/latest/?badge=latest)
[![License: GPL v2](https://img.shields.io/badge/License-GPL_v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)

**PLEASE NOTE:** The code is currently under active development and is still in beta versions 0. x.x.

## Requirements
- [Python](https://www.python.org/) 3.7  or later
- [NumPy](https://numpy.org/doc/stable/reference/)
- [ase](https://wiki.fysik.dtu.dk/ase/)
- [NetworkX](https://networkx.org/)
- [pandas](https://pandas.pydata.org/)
- [yaml](https://pyyaml.org/)

## Installation
Clone Directory
~~~bash
git clone https://github.com/kkjsawantucla/gg.git
~~~

Install using pip
~~~bash
cd gg/
pip install .
~~~

Alternatively, you can add ./gg to your $PYTHONPATH. (not recommended)
~~~bash
export PYTHONPATH=$PYTHONPATH:"<path_to_gg>"
~~~

## Usage

### GCBH
The Gcbh calculator is an ase Dynamics Child on unhealthy steriods. It runs the grand canonical basin hopping, however certain functionalities are hard coded.

#### Inputs
1. atoms (ase.Atoms): An [ase Atoms](https://wiki.fysik.dtu.dk/ase/ase/atoms.html) object as a starting point. The object should have a [ase.calculator](https://wiki.fysik.dtu.dk/ase/ase/calculators/calculators.html) attached. 
2. logfile (str): path to a file that logs the calculator's output.
3. trajectory (str): path to a file that logs all the atoms structure files visited by the calculator.
4. config file (str): A yaml file that takes specific inputs for the Gcbh calculaor. In the future, more functionalities will be read from the config file. Please check the example folders to check the currently available functionalities.
5. restart (bool): To control restart from previous calculations.
6. optimizer (ase.optimizer): An [ase Optimizer](https://wiki.fysik.dtu.dk/ase/ase/optimize.html) that controls geometric optimization of a given ase.atoms object and reduce forces. The default is BFGS.

~~~bash
from gg.gcbh import Gcbh
from ase.io import read
from ase.calculators.emt import EMT

atoms = read('POSCAR')
atoms.calc = EMT()
G = Gcbh(atoms,config_file='input.yaml')
~~~

#### Modifiers
The modifiers form the building block of the code. They determine how the atoms are modified during each basin hopping step. The code provides basic modifiers as building blocks for more complex modifiers.

##### 0. Sites
The user need to specify a Site class so that the modifier knows which specific atoms to work on when an ase.Atoms object is provided. Here we show an example of Surface Site class which uses the co ordination number of each element as way to recognize surface atoms.

~~~bash
from gg.sites import SurfaceSites

max_coord = {"Pt": 12, "O": 2, "H": 1} #The max coord of each element in the atoms object
ss = SurfaceSites(max_coord, max_bond_ratio=1.2) #The max_bond_ratio is needed to make graph
~~~

##### 1. Add Modifier
The modifier can add an adsorbate, or moiety at specific sites on the parent atoms object.

~~~bash
from gg.modifiers import Add

adsorbate_OH = read("OH.POSCAR") #adsorbate to be added
add_OH = Add(weight=1, ss, ads=adsorbate_OH, surf_coord=1, ad_dist="O", surf_sym=["Pt"])
G.add_modifier(add_OH,'Add_OH') #add the modifier to Gcbh and give it a name for identification
~~~

##### 2. Remove Modifier
The modifier can remove an adsorbate, or moiety at specific sites on the parent atoms object.

~~~bash
from gg.modifiers import Remove

~~~

##### 3. Swap Modifier

##### 4. Combining Modifiers

### Running Gcbh
~~~bash
G.run(steps=10)
~~~
The code will generate a folder called "opt_folder" containing individual geometric optimization runs. Additionally, it will log the details of the gcbh run in "gcbh.log" and the trajectory of atoms visited at "gcbh.traj". The run will also dump the status of the run in "current_status.pkl", which will be useful in restarting a calculation. Finally, it will log the accepted structures in "local_minima.traj"
