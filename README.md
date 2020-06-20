# SVMRanker - ESEC/FSE Tool Demo 2020
# Review ID - 21

This repository is prepared for reviewing SVMRanker, which is submitted to [ESEC/FSE 2020 Tool Demo](https://2020.esec-fse.org/track/esecfse-2020-tool-demos#Call-for-Tool-Demos).
SVMRanker utilizes SVM techniques to synthesize multiphase ranking functions for proving loop programs.

## Source Code
**SVMRanker**: [https://github.com/ESEC-FSE-2020-Tool-Demo-ID21/SVMRanker](https://github.com/ESEC-FSE-2020-Tool-Demo-ID21/SVMRanker).

## Benchmarks
Out benchmark of 225 programs which consist of 134 boogie programs and 91 programs from C is here: []()


## Log files of the experiments

## Screencast of the tool demo

The screencast of the demo is available [HERE]()

## SVMRanker
### Installation
You should have installed Python 3 and Java Development Kit on your system.
Currently we can successfully run SVMRanker with Python 3 and JDK 8.0.

**Install Python packages**

```
pip3 install z3-prover
pip3 install click
pip3 install sklearn
pip3 install python-constraint
```

### Usage

After having installed the required software, SVMRanker can be used by entering the *src/* directory and then calling SVMRanker as follows: 
```
  python3 ./CLIMain.py --help
```
You should be able to see the following output.
```
```
As we can see, SVMRanker provides five commands. 
The first two commands allow for proving termination of a given program while the remaining three can be used for parsing the input file and translate it to a different format. 
In the remaining part of the section we focus on the details for the use of the **lmulti** and **lnested** commands.

**lmulti**, short for learning multiphase ranking function, instructs SVMRanker to learn a multiphase ranking function for the given program. 
To get the detailed usage information for this command, one can use the following command.
```
  python3 ./CLIMain.py lmulti --help
```
As the help shows, there are several options available to tune the execution of **lmulti**; 
we present their usage by means of a couple of examples. 
```
//example/Example1.c
	int main() {
    	int x, y;
    	while(x > 0 || y > 0) {
    		x = x + y - 1;
    		y = y - 1;
    	}
	}

```

The is the first C program we consider here; see the file **src/example/Example1.c**. 
This program can be shown to be terminating by means of a 2-multiphase ranking function, as we get by running SVMRanker to learn its multiphase ranking function as follows.
```
  python3 ./CLIMain.py lmulti --filetype C example/Example1.c
```
SVMRanker completes the analysis by returning a 2-multiphase ranking function for the **Example1.c** program, as shown below.
```
```
Notice that we used the option **--filetype** to specify the type of the input program, given that SVMRanker supports both Boogie programs and C programs as input file, with the former being the default format. 
Furthermore, we can also provide the option **--depth_bound** to set the maximal number of phases SVMRanker can use when learning a multiphase ranking function. 
The default value of this option is 2, and this has been enough in the analysis of **Example1.c**, since **Example1.c** can be proved to terminate by a 2-multiphase ranking function. 
Such a default value is suitable for several programs, but it can be increased as needed, as we will see with the second example **Example2.c**, as shown below.
```
	//example/Example2.c
	int main() {
    	int x, y, z;
    	while (x > 0) {
    		x = x + y;
    		y = y + z;
    		z = z - 1;
    	}	
	}
```
If we run SVMRanker on **Example2.c** with the default value of 2 for **--depth_bound**, we obtain **Unknown** as result. 
In order to get an appropriate multiphase ranking function for **Example2.c**, one needs to call **lmulti** with the option **--depth_bound** set to at least 3.
```
  python3 ./CLIMain.py lmulti --filetype C --depth_bound 3 example/Example2.c
```
With the help of **--depth_bound**, SVMRanker produces the result shown below.
```
```
We now present the other options that let SVMRanker use different strategies in the process of learning a multiphase ranking function; 
different strategies regarding how program data points are sampled, how the state space is cut, and what templates are used, influence the running time of SVMRanker and possibly the final result. 

* **--sample_strategy**
	This option controls the strategy SVMRanker uses to sample program data points.
	Possible values are *CONSTRAINT* and *ENLARGE* (the default):
	*CONSTRAINT* samples randomly the points satisfying the loop condition/guard;
	This strategy can be slow as we need to get the assignments of variables satisfying the loop guard. 
	*ENLARGE* samples uniformly the points in a predefined space, which can be very efficient; However, if all the sampled points cannot satisfy the loop guard, we will then try to enlarge the sampling space until seeing some point satisfying the loop guard.

* **--cutting_strategy**
	This option controls the bound $b$ of the constraint f(x) < b that is used to cut the program state space in two parts for the current phase's decreasing function f.
	Possible values are *NEG*, *POS*, and *MINI* (the default):
	*NEG* chooses randomly a negative value for b;
	*POS* chooses randomly a positive value for b;
	*MINI* selects the minimum value of f on the sampled points.
* **--template_strategy** 
	This option controls what templates are used in the learning procedure.
	Possible values are *FULL* and *SINGLEFULL* (the default):
	*FULL* uses as templates the linear combinations of all program variables; 
	*SINGLEFULL* extends FULL with templates using only one variable at a time.

* **--print_level*
	This option controls the verbosity of the SVMRanker output.

