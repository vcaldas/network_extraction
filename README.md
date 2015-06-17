Disclaimer: The scripts and libraries uploaded in this project are intended to be a suite of tools for the extraction, manipulation and analysis of network data (graphs) from images. The scripts uploaded here are part of a methods paper detailing the algorithms used which is soon to be published.
The original images used as examples in the paper can be found in the /data directory. The images represent a range of different use-cases for NEAt both from different research projects as well as with regards to the size of the networks they contain. They are ready to be processed without any further modifications.

# Setting up the NEAT framework
The neat framework is a set of python scripts that make use of the python module *neat_helpers* and the cythonized library *C_neat_functions*. For neat to work you will need three things
1. a working installation of python 2.7 and pip
2. the third party libraries utilized within NEAT (scipy, numpy, matplotlib, cython, networkx, scikit-image, opencv, pillow, shapely, meshpy)
3. a version of the C_neat_functions.pyx file compiled for your platform
In the following we detail how to get these three things on Linux, Windows and MacOS platforms.

## Linux
1. Python 2.7 should be installed by default, if it is not, your package manager most likely will provide it.
	 You can get pip by running  
	```
	wget https://bootstrap.pypa.io/ez_setup.py -O - | sudo python
	```
	 (More detailed instructions on how to get pip can be found [here](https://pypi.python.org/pypi/setuptools#unix-wget)).
 
2. For numpy (and some other libraries) you will need the python-dev package, get it by running
	```
	sudo apt-get install python-dev 
	```
	Now you have pip, it should be easy to install most of the other libraries. For example to install networkx, simply run
	```
	pip install networkx
	```
	Repeat this process with scipy, matplotlib, cython, scikit-image, pillow, meshpy and shapely. 
	To install opencv, follow the instructions detailed [here](http://docs.opencv.org/doc/tutorials/introduction/linux_install/linux_install.html).

3. To compile your own *C_neat_functions.so* file, navigate to the /neat directory and run
	```
	python setup_C_neat_functions.py build_ext --inplace
	```
	This should create two files called *C_neat_functions.so* and *C_neat_functions.c*, the first you need, the latter you can safely delete.

--------------------------------------------------

## Windows
1. Download the latest python 2.7 release [here](https://www.python.org/downloads/) and install it.
	To get pip, if you are on windows 8 or later, start Powershell and run
	```
	(Invoke-WebRequest https://bootstrap.pypa.io/ez_setup.py).Content | python -
	```
	If you have an earlier windows version, download *ez_setup.py* [here](https://bootstrap.pypa.io/ez_setup.py),
	navigate to the directory you saved the file in and run
	```
	python ez_setup.py
	```
	(More detailed instructions on how to get pip can be found [here](https://pypi.python.org/pypi/setuptools#unix-wget)).

2. Thankfully, Christoph Gohlke maintains a site with a wide collection of python packages for windows. Go to
	[his web page](http://www.lfd.uci.edu/~gohlke/pythonlibs/) and download the .whl files for scipy, numpy, matplotlib, cython, networkx, scikit-image,
	 opencv, pillow, meshpy and shapely that fit your platform. Install the packages by navigating to the directory you saved them in and run
	```
	pip install SomePackage.whl
	```
	for all the packages you downloaded.

3. Navigate to the folder /neat and run
	```
	python setup_C_neat_functions.py build_ext --inplace
	```
	A file named *C_neat_functions.pyc* should have bean created in the \neat directory
	Compilation might be abortet with the message "error: Unable to find vcvarsall.bat"
	There is a relatively easy fix for that (based on [this stack overflow post](http://stackoverflow.com/questions/2817869/error-unable-to-find-vcvarsall-bat)):
	Install Visual studio 2013 and set the environment variable by running
	```
	SET VS90COMNTOOLS=%VS120COMNTOOLS% 
	```
	Rerun
	```
	python setup_c_neat_functions.py build_ext --inplace
	```
	If it still does not work, good luck!

--------------------------------------------------

## Mac OSX
1. Download and install the latest python 2.7 release [here](https://www.python.org/downloads/) and install it.
	Depending on whether you have access to easy_install or curl, run one of the following
	```
	sudo easy_install pip
	curl https://bootstrap.pypa.io/ez_setup.py -o - | python
	```
	(More detailed instructions on how to get pip can be found [here](https://pypi.python.org/pypi/setuptools#unix-wget)).

2. For numpy (and some other libraries) you will need the python-dev package, get it by running
	```
	sudo apt-get install python-dev 
	```
	Now you have pip, it should be easy to install most of the other libraries. For example to install networkx, simply run
	```
	pip install networkx
	```
	Repeat this process with scipy, matplotlib, cython, scikit-image, pillow, meshpy and shapely. 
	To install opencv, follow the instructions detailed [here](http://tilomitra.com/opencv-on-mac-osx/).

3. Navigate to the folder /neat and run
	```
	python setup_C_neat_functions.py build_ext --inplace
	```
	This should create two files called *C_neat_functions.so* and *C_neat_functions.c*, the first you need, the latter you can safely delete.

________________________________________________________

# Extracting network data using the NEAT framework 
The network extraction and analysis tool (NEAT) is intended for the extraction of network data from images which can later be analyzed easily.
The workflow is broken down into four steps represented by four processing scripts

1. Segment the original image into foreground and background and therefore create a binary image using binarize.py
2. Extract the network from the binary image unsing neat.py
3. Optional: manually correct errors in the network or remove artifacts using the graph-edit GUI gegui.py
4. Analyze basic characteristics of the network with analyze.py

## binarize
To get from a grayscale to a suitable binary image can be tricky. I have provided rudimentary scripts which do some contrast improvement and thresholding for different kinds of images (*binarize_tracheole.py* for tracheoles, *binarize_crack.py* for cracks and *binarize_leaf.py* for leaves). 
All scripts take the path to the image as required arguments as well as some optional arguments to modify their behaviour.
Imput images can be of several formats (read the [PIL Image file formats documentation](http://pillow.readthedocs.org/en/latest/handbook/image-file-formats.html) for reference). Common formats like *.png*, *.jpg* and *.tif* are supported in any case.

### Tracheoles and Cracks
The two scripts behave similarly with regards to their processing methods. I have separated the scripts nevertheless to improve clarity and enable diversification later on. Available parameters are

- **-g** kernel size of the gaussian blur applied to reduce noise in the image at the beginning. *g* should be significantly smaller than the smallest structure in the image you want to resolve. *If g = 0*, no gaussian blur will occur.
- **-t** value for the threshold used for dividing the image into background and foreground (black and white). E.g. if *t = 100*, all pixels with a value smaller or equal t will be mapped to zero (black) and all pixels larger than t will be mapped to one (white). t should be choosen in a way that all important structures are present and connected while everything that belongs to the background vanishes.
- **-s** kernel size of the binary opening and closing operators applied to smoothen the contours of the foreground. *s* should be considerably smaller than the diameter of the thinnest network part. If *s* is too high, information is lost as foreground structures vanish. If s is too small, the operation has close to no effect. If *s = 0*, no smoothing will occur.
- **-m** minimum size of features (in pixels) that will be preserved in the image. This will remove artifacts and noise that are disconnected from the main network. If for example *m = 5000*, structures composed of less than 5000 pixels will be removed from the image. In general, I advise to set *m* smaller than 1/10th of the total number of pixels in the image. If m is choosen to high, the main structure might vanish. If *m* is choosen too low, atrifacts and noise might remain in the image.

To segment an image of a tracheole (for example *tracheole1.png*) with parameters *g = 0* (no blurring), *t = 110*, *s = 3* and *m = 500*, run

```
python binarize_tracheole.py ../data/originals/tracheole1.png -g 0 -t 110 -s 3 -m 500
```

To segment an image of a crack pattern (for example *cracks2.png*) with parameters *g = 3*, *t = 150*, *s = 2* and *m = 10000*, run

```
python binarize_crack.py ../data/originals/cracks2.png -g 3 -t 150 -s 2 -m 10000
```

### Leaves
This script behaves a bit differently as in addition to thresholding and smoothing it also tries to increase the contrast in the image. Available parameters are

- **-g** as before: the kernel size of the gaussian blur operation.
- **-eq** the kernel size of the local histogram equalization applied to increase contrast. If *eq = 0*, no local histogram equalization will occur.
- **-r** After local histogram equalization has been applied, a fraction of the processed image is recombined with a fraction of the original image. *r* determines said fraction. Therefore if *r = 0.3*, the outcome of the recombination process will be *0.3 x equalized image + 0.7 x original image*.
- **-o** The threshold for thresholding the image to create a binary image is determined automatically using Otsu's method. *o* controls an offset to the automatically determined threshold. Therefore if *o = 10* and the threshold calculated via Otsu's method equals 125, the image will be thresholded with a threshold *t = 135* (also allows for negative values of *o*).
- **-s** as before: kernel size of the binary opening and closing operators for smoothing.
- **-m** as before: minimum feature size.

To segment an image of a leaf venation pattern (for example *leaf1.png*) with parameters *g = 5*, *eq = 11*, *r = 0.5*, *o = 25*, *s = 3* and *m = 50000*, run

```
python binarize_leaf.py ../data/originals/leaf1.png -g 5 -eq 11 -r 0.5 -o 25 -s 3 -m 50000
```

--------------------------------------------------

## neat
Extraction of network data is done with the *neat.py* script. NEAT takes the path to a binary (black and white) image as required command line argument and extracts a graph (networkx graph-object) from the largest connected component in the image. The format of the binary image needs to be pixel based (vector graphics need to be converted first) but other than that, [all formats supported by the PIL library](http://pillow.readthedocs.org/en/latest/handbook/image-file-formats.html) are possible.
NEAT's behaviour can be modified by several parameters and switches:
- **-source** Required argument. The path to the image you want to process, for example *../data/originals/tracheole3.png*. Either a relative path to the processing script or a total path works.
- **-help** displays all available options with a short description in the command line.
- **-dest** (default = **source**) If you want to save NEAT's results in a different folder than **source**, specify the **dest** argument. If for example you want to process *leaf1.png* from the */data/originals* folder but save the resulting network in the */neat* folder, specify **-dest** *../neat/*. Either a relative path to the processing script or a total path works.
- **-v** (default = False) NEAT is run via the command line but by default it will not produce any command line output to avoid spam. If you enable verbosity by specifying **v**, NEAT will report on its current processing state and output some information on how long it took the script to complete the previous processing step.
- **-d** (default = False) If something goes wrong, NEAT crackes or the output is not what you expected, it might help to enable the debugging mode by specifying **d**. This will further increase the verposity of the script. Moreover, NEAT will save visualizations of intermediate processing steps so you can have a look and maybe figure out what is going wrong. The visualizations include a plot of the extracted contours, the triangulation and an overlay of the contours, triangulation, distance map and extracted network.
- **-plt** (default = False) Plotting and saving images (especially really large ones like NEAT is accustomed to dealing with) takes a lot of time. This is why by default visualization of the extracted networks is disabled. If you want to see what NEAT extracted, specify **plt**. By default, NEAT will save plots in *.pdf* format. This can take ages for graphs with more than ~1000 nodes. If you want to change the plot file format, specify the **fformat** argument.
- **-fformat** (default = *.pdf*) File format of the plots if **plt** is enabled. To display all available formats for your platform, start a python interpreter (for example by typing 'python' in your terminal) and run

```python
import matplotlib.pyplot as plt
plt.gcf().canvas.get_supported_filetypes()
```
... If you want do visualize a really large graph, I would recommend to set **fformat** to *.png* and crank up the resolution of the image by specifying **dpi** so details are still recognizeable.
- **-dpi** (default = 500) If saving plots in non-vector-graphics formats like *.png* or *.jpg* you can change the resolution by specifying the **dpi** argument. 
- **-gformat** (default = *.gpickle*) If you want to change the format the extracted graph is saved in, specify the **gformat** argument. For a list of supported formats, have a look at the [networkx reading and writing graphs documentation](https://networkx.github.io/documentation/latest/reference/readwrite.html). The default *.gpickle* format is a dump of a python object (in this case a networkx.Graph object). It is handy as all our processing is done in python and it can easily be read again to be the same python object it was before saving to the harddrive. For a cross-program format *.gml* most likely is the best choice as it is widely used and simple.
- **-r** (default r = 0) During the extraction process, the graph is made up of many points originally stemming from the triangles making up the contstrained delaunay triangulation at the heart of the vectorization process. Most of these points are not important for the topology of the graph (we therefore call them "redundant") as they lie on long stretches of the network that do neither represent junctions nor tips. These points are only there to support the geometry of the network. By specifying **r** you can decide how many of these redundant points you want to keep in your final representation. If *r = 0* only nodes and tips will be saved and all redundant points will be discarded - this is nice if you don't care about geometry and want to save some time and disk space. If *r = 1*, approximately half of the redundant points will be kept. This is a good compromise between an acceptable approximation of the network's geometry and speed/size. If *r = 2*, all redundant points will be saved. This produces large files and the resulting graphs might take some time to load and analyze but it also has by far the best approximation of the network's geometry.
- **-p** (default p = 3) If the edges of your binary image are noisy, jagged or especially wiggly (which is the case for most images of biological networks), this might lead to the emergence of spurious branches in the extracted networks. These branches are extremely short and therefore easily recognizeable. By pruning away all branches that are shorter than **p** points (see the definition of redundant points in the description of the **r** parameter), we get rid of spurious branches. Treat **p** with care as you might lose information if **p** is too high (genuine branches are pruned away) and if **p** is too low, spurious branches might remain. **p** should be choosen considerably smaller than the average edge length, commonly between 2 and 6.
- **-m** (default m = False) If you are given a binary image that was maybe not processed with one of the *binarize* scripts and with whose quality you are not satisfied, you can use the **m** parameter, to remove disconnected artifacts and noise from the image (as described in the *binarize* section).
- **-s** (default s = False) As with the **m** parameter, by specifying **s** you can do some improvement of your binary image by smoothing contours before you start processing (as described in the *binarize* section).
- **-dm** (default dm = False) If you want to save the distance map that is created during the graph extraction process, enable **dm**. If you want to later open and edit the graph with *GeGUI*, you will need the distance map so better save it right away, it will be created anyways.

If you want to extract a graph from the image */data/binaries/tracheole6.png*, set pruning to 5, save all redundant nodes, enable verbosity and additionally save a visualization of the graph in *.png* format with a resolution of 2000 dpi, run
```
python neat.py ../data/binaries/tracheole6.png -p 5 -r 2 -v -plt -fformat png -dpi 2000
```
If you have a shitty binary that you want to smooth and clear up (like */data/binaries/tracheole8.png*), you want no pruning, are content with only some redundant nodes, want to have a visualization of intermediate steps, verbosity and a distance map but no plot of the final graph, then run
``` 
python neat.py ../data/binaries/tracheole8.png -s 2 -m 1000 -p 0 -r 1 -d -v -dm
```

--------------------------------------------------

## gegui
I have been dealing with networks extracted from pseudo 2D structures a lot (especially the tracheole dataset). These images are projections onto a plane and therefore might contain network "crossings" which arent real but just created by superposition of two branches.
To correct these spurious junctions I have written the *GeGUI* (graph edit GUI) which allows you to load the extracted graph object, superimpose it onto the original image and modify the graph's structure by manually deleting and creating nodes and edges.
The script to start the GUI is called *gegui.py* but all the functionality is placed in the three sub-scripts *InterActor.py* (dealing with user-interaction), *GraphHandler.py* (dealing with manipulations of the graph object) and *PlotHandler.py* (dealing with the dynamic display of changes to the graph on the screen). To run the *GeGUI* you need to specify a folder in which the graph you want to process is located. For the GeGUI to properly start the folder also needs to contain an euclidean distance map of the binary the graph was extracted from and the original image. Therefore the folder should look like this:

```
/resultdir
----original_image.png
----extracted_graph_red1.gpickle
----distance_map_dm.png
```

I the */data/results* folder I have prepared several such folders (e.g. *tracheole1*, *tracheole2* and *tracheole3*) to test *GeGUI*. If you want to load and process for example the graph extracted from *tracheole1.png*, run
```
python gegui.py ../data/results/tracheole1
```

--------------------------------------------------

## analyze


### TODO
- sort NEAT's options in a comprehensible way
- refer to the methods paper as soon as it is in the ArXiv
- implement an option to modify the node size in plots
- specify how GeGUI selects the right files in a folder
- describe the analyze script
- add markdown example processings for binarize, neat and maybe also analyze including images
- maybe add screenshots to geguis description



