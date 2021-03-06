# Overview

The datamining pipeline is a part of SG++ that provides a streamlined way to
do datamining tasks such as regression, density estimation and classification
with sparse grids.<br/>
The pipeline intends to reduce the amount of code that needs to be written
for a SG++ executable and instead tries to move as much configuration and set-up as possible to
JSON configuration files. As such the pipeline may even be used out-of-the-box
without any programming required.<br/>

The configuration of the datamining pipeline relies on plain JSON configuration files
which are provided as an input to a program running the pipeline. The program
then parses the configuration and proceedes with the datamining as specified by
the configuration.

The top-level structure of pipeline configuration is structured as follows:


```
example_config.json:
{
    "dataSource": {
        ...
    },
    "scorer": {
         ...
     },
    "fitter": {
        ...
     },
 }
```

Each of these three attributes has many different sub-attributes to configure.
Note that almost all attributes have built-in default values and it is often enough
to only specifiy a select few attributes manually in the JSON configuration file.

In the following all available attributes are listed in tablular form. The three
top-level JSON attributes are <b>dataSource</b>, <b>scorer</b> and <b>fitter</b>.
Each of these may have attributes which are JSON dictonaries themselves. These are then
listed separately.<br/>
The column "Depends on" lists attribute-value pairs that the row in questions
depends on. If the condition in the "Depends on" column is
not fullfilled, the attribute in question will have no effect (but may still
be set - it is just ignored by the program).


# Data Source Configuration

This attribute contains a JSON dictionary that specifies the input data.
For instance where the datafile is located, if the data contains targets
(for supervised learning) or what columns of the data should be considered for
read-in.

### dataSource
<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>filePath</td><td>String</td><td>relative or absolute path</td><td>Path to the datafile. Path may be absolute or relative to the directory of execution of the binary</td><td></td></tr>
<tr><td>fileType</td><td>String</td><td>"arff", "csv", "none"</td><td>The file type of the datafile. Currently supported are ARFF and CSV files as well gzip compressed file. If "none" is supplied the file type is automatically inferred from the file extension. Note independent from the configuration file: Files used with the pipeline <b>must</b> have one of these extensions (with optional ".gz" extension for compressed files), otherwise the datafile cannot be read</td><td></td></tr>
<tr><td>compression</td><td>Boolean</td><td>true, false</td><td>Supply true if the file containing the data is gzip compressed. It will then be automatically decompressed as the data is read in by the pipeline</td><td></td></tr>
<tr><td>numBatches</td><td>Integer</td><td>[1, inf)</td><td>Into how many batches the dataset should be split. If the value is 1 then the entire dataset is taken as a whole</td><td></td></tr>
<tr><td>batchSize</td><td>Integer</td><td>[0, inf)</td><td>The size of one batch. If 0 then then take all avaliable samples</td><td></td></tr>

<tr><td>validationPortion</td><td>Float</td><td>[0.0, 1.0]</td><td>The fraction of the available data to be used as validation data</td><td></td></tr>
<tr><td>hasTargets</td><td>Boolean</td><td>true, false</td><td>If true the last column of the data file is treated as targets for supervised learning. If this is the case then this target column is not part of the data per se, i.e. it will not be a column in the sgpp::base::DataMatrix containing all data but rather in a sgpp::base::DataVector containing the targets</td><td></td></tr>
<tr><td>shuffling</td><td>String</td><td>"sequential", "random"</td><td>How the data is shuffeled. Sequential means no shuffeling</td><td></td></tr>
<tr><td>randomSeed</td><td>Integer</td><td>(-inf, inf)</td><td>The seed used for all (most, excluding HPO) random number generation used by the pipeline</td><td></td></tr>
<tr><td>epochs</td><td>Integer</td><td>[1, inf)</td><td>The number of epochs to train</td><td></td></tr>
<tr><td>readinCutoff</td><td>Integer</td><td>[-1, inf)</td><td>The row-index in the dataset after which to cut off the data read in.
The value -1 indicates that all rows should be read in</td><td></td></tr>
<tr><td>readinClasses</td><td>Float List</td><td>Subset of class labels</td><td>Only rows with a class label in this list are read in. An empty list means all class labels are considered. This is intended for class labels and floats are equal with 10e-3 precision</td><td>hasTargets=true</td></tr>
<tr><td>readinColumns</td><td>Integer List</td><td>Subset of column indices</td><td>Only columns with index in this list are read in. An empty list means all columns are read in</td><td></td></tr>
<tr><td>dataTransformation</td><td>Dictionary</td><td>see table below</td><td>Configuration for further data transformation, i.e. rosenblatt transformation</td><td></td></tr>
</table>


The dataTransformation is an attribute to transform the data <b>after</b> it is read in. Currently only rosenblatt transformation is
implemented but this could be used for normalization, outlier-removal, etc. in the future.

### dataTransformation
<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>type</td><td>String</td><td>"rosenblatt", "none"</td><td>The type of data transformation. Currently only rosenblatt transformation is available</td><td></td></tr>
</table>

# Scorer Configuration

The scorer evaluates the model during training with respect to a metric that is
specified in this attribute.
Currently implemented are mean squared error (MSE), accuracy (L2 norm of the 
difference between prediction and targets), and negative log likelihood (NLL).

### scorer
<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>metric</td><td>String</td><td>"mse", "nll", "accuracy"</td><td>The metric that the scorer uses during learning</td><td></td></tr>
</table>

# Fitter Configuration

The fitter configuration specifies what and how the datamining task should be
performed. Notably, the type of the fitter will specifiy the task, currently
either one of least square regression, density estimation or classification.<br/>
Besides the type, many aspects of the SG++-related parameters, such
as the grid type, grid level (gridConfig),
refinement and coarsening behavior (adaptivityConfig) and much more may be configured.

### fitter
<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>type</td><td>String</td><td>"regressionLeastSquares", "densityEstimation", "classification"</td><td>Currently "classification" is not implemented in the UniversalMiner (see Fitter configuration below for more info)</td><td></td></tr>
<tr><td>gridConfig</td><td>Dictionary</td><td>see table below</td><td></td><td></td></tr>
<tr><td>adaptivityConfig</td><td>Dictionary</td><td>see table below</td><td></td><td></td></tr>
<tr><td>crossValidation</td><td>Dictionary</td><td>see table below</td><td></td><td></td></tr>
<tr><td>densityEstimationConfig</td><td>Dictionary</td><td>see table below</td><td></td><td>type="densityEstimation" or type="classification"</td></tr>
<tr><td>database</td><td>Dictionary</td><td>see table below</td><td></td><td></td></tr>
<tr><td>solverRefineConfig</td><td>Dictionary</td><td>see table below</td><td></td><td></td></tr>
<tr><td>solverFinalConfig</td><td>Dictionary</td><td>see table below</td><td></td><td></td></tr>
<tr><td>regularizationConfig</td><td>Dictionary</td><td>see table below</td><td></td><td></td></tr>
<tr><td>learner</td><td>Dictionary</td><td>see table below</td><td></td><td></td></tr>
<tr><td>parallelConfig</td><td>Dictionary</td><td>see table below</td><td></td><td></td></tr>
<tr><td>geometryConfig</td><td>Dictionary</td><td>see table below</td><td></td><td></td></tr>
</table>

The grid configuration allows to specify general grid properties such as the
grid type, the initial level, etc. Note, that the dimension can usually
be inferred automatically from the dataset which overwrites
the dim attribute listed here.

### gridConfig
<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>gridType</td><td>String</td><td>all sgpp::base::GridTypes</td><td>The type of the grid</td><td></td></tr>
<tr><td>dim</td><td>Integer</td><td>[1, inf)</td><td>The dimension of the grid</td><td></td></tr>
<tr><td>level</td><td>Integer</td><td>[1, inf)</td><td>The level of the grid (initial, before refinement)</td><td></td></tr>
<tr><td>generalGridType </td><td>String</td><td>"regular", "component"</td><td>"regular": compute solution on one sparse grid, "component": use the combination technique (works only, if the library is compiled with USE_PYTHON_EMBEDDING=1)</td><td></td></tr>
<tr><td>maxDegree</td><td>Integer</td><td>[1, inf)</td><td>The maximal degree of polynomials used if the grid uses polynomial basis functions</td><td>gridType uses polynomial ansatz functions</td></tr>
<tr><td>boundaryLevel</td><td>Integer</td><td>[0, inf)</td><td>The level of the boundary grid. <b>This parameter is currently recommended to be left empty.</b></td><td>gridType is a boundary grid</td></tr>
<tr><td>fileName</td><td>String</td><td></td><td>The file name used if the grid is to be serialized to a file</td><td></td></tr>
</table>

The adativity configuration allows to specify parameters related to adaptive grid refinement
and coarsening.

### adaptivityConfig
<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>numRefinements</td><td>Integer</td><td>[0, inf)</td><td>The number of refinements</td><td></td></tr>
<tr><td>threshold</td><td>Float</td><td>[0.0, inf)</td><td>Threshold for surplus refinement. Only grid points with a surplus above this threshold are considered for refinement</td><td></td></tr>
<tr><td>maxLevelType</td><td>Boolean</td><td>true, false</td><td>The refinement type. Currently unused</td><td></td></tr>
<tr><td>noPoints</td><td>Integer</td><td>[0, inf)</td><td>The maximal number of points to refine during one refinement</td><td></td></tr>
<tr><td>percent</td><td>Float</td><td>[0.0, 1.0]</td><td>The maximal fraction ("percent") of points to be refined</td><td></td></tr>
<tr><td>refinementIndicator</td><td>String</td><td>"surplus", "surplusVolume", "zeroCrossing", "dataBased", "gridPointBased", "multipleClass"</td><td>The functor type use for refinement</td><td></td></tr>
<tr><td>precomputeEvaluations</td><td>Boolean</td><td>true, false</td><td>Determines if evaluations should be pre-computed during zero-crossing-based refinement</td><td>refinementIndicator="zeroCrossing"</td></tr>
<tr><td>penalizeLevels</td><td>Boolean</td><td>true, false</td><td>Should grid points with higher level get penalized when refinement candidates are picked?</td><td></td></tr>
<tr><td>scalingCoefficients</td><td>[Float]</td><td>List of length #classes with values (0, inf)</td><td>The scaling coefficients alpha for each class which are part of data-based refinement</td><td>refinementIndicator="dataBased"</td></tr>
<tr><td>errorBasedRefinement</td><td>Boolean</td><td>true, false</td><td>errorbasedRefinement is a refinement strategy that observes how effective additional training with the same grid is and decides for further refinement only if the improvement declines. If false, then refinement is done after refinementPeriod number of data points where applied to the model / learning process</td><td></td></tr>
<tr><td>refinementPeriod</td><td>Integer</td><td>[0, inf)</td><td>This values determines the period of refinements in relation to the number of data points used in the learning process. A period of 200 means, that every 200 data points the grid is refined.</td><td>errorBasedRefinement=false</td></tr>
<tr><td>errorMinInterval</td><td>Integer</td><td>[0, inf)</td><td>Minimum amount of iterations before the next refinement is allowed to happend</td><td>errorBasedRefinement=true</td></tr>
<tr><td>errorBufferSize</td><td>Integer</td><td>[0, inf)</td><td>Amount of error values to consider when checking for convergence</td><td>errorBasedRefinement=true</td></tr>
<tr><td>errorConvergenceThreshold</td><td>Float</td><td>(0, inf)</td><td>Threshold for convergence</td><td>errorBasedRefinement=true</td></tr>
</table>

Configuration for crossvalidation. To make use of crossvalidation we need a portion of the data
dedicated to that purpose. The attribute to control this cross validation data is
in the configuration dataSource "validationPortion".

### crossValidation
<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>enable</td><td>Bool</td><td>true, false</td><td>Should crossvalidation be used?</td><td>dataSource.validationPortion > 0.0</td></tr>
<tr><td>kFold</td><td>Integer</td><td>[1, inf)</td><td>Number of batches for used during cross validation</td><td>enable=true</td></tr>
<tr><td>randomSeed</td><td>Integer</td><td>[0, inf)</td><td>Seed for determining the k-fold</td><td>enable=true</td></tr>
<tr><td>shuffle</td><td>Bool</td><td>true, false</td><td>Should the k-fold be shuffled, i.e. randomized (true) or left sequential (false)?</td><td>enable=true</td></tr>
<tr><td>silent</td><td>Bool</td><td>true, false</td><td>Should there be verbose output?</td><td>enable=true</td></tr>
<tr><td>lambda</td><td>Float</td><td>[0, inf)</td><td>Regularization parameter</td><td>enable=true</td></tr>
<tr><td>lambdaStart</td><td>Float</td><td>[0, inf)</td><td>Lower bound of lambda search range</td><td>enable=true</td></tr>
<tr><td>lambdaEnd</td><td>Float</td><td>[0, inf)</td><td>Upper bound for lambda search range</td><td>enable=true</td></tr>
<tr><td>lambdaSteps</td><td>Integer</td><td>[1, inf)</td><td>The number of steps between [lambdaStart, lambdaEnd] to be tested</td><td>enable=true</td></tr>
<tr><td>logScale</td><td>Bool</td><td>true, false</td><td>Should the interval [lambdaStart, lambdaEnd] be interpreted as logscale?</td><td>enable=true</td></tr>
</table>

The density estimation configuration is only relevant if the fitter (type) is densityEstimation or classification which
is a sub-type of densityEstimation.

### densityEstimationConfig
<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>densityEstimationType</td><td>String</td><td>"cg", "decomposition"</td><td>Should conjugate gradient or matrix decomposition be used?</td><td></td></tr>
<tr><td>normalize</td><td>Bool</td><td>true, false</td><td>Should the density
function be normalized?</td><td>fitter.densityEstimationType="decomposition"</td></tr>
<tr><td>matrixDecompositionType</td><td>String</td><td>"cg", "eigen", "chol", "denseichol", "orthoadapt"</td><td>The type of matrix decomposition. "cg" means LU decomposition</td><td>densityEstimationType="decomposition"</td></tr>
<tr><td>iCholSweepsDecompose</td><td>Integer</td><td>[0, inf)</td><td></td><td>matrixDecompositionType="denseichol"</td></tr>
<tr><td>iCholSweepsRefine</td><td>Integer</td><td>[0, inf)</td><td></td><td>matrixDecompositionType="denseichol"</td></tr>
<tr><td>iCholSweepsUpdateLambda</td><td>Integer</td><td>[0, inf)</td><td></td><td>matrixDecompositionType="denseichol"</td></tr>
<tr><td>iCholSweepsSolver</td><td>Integer</td><td>[0, inf)</td><td></td><td>matrixDecompositionType="denseichol"</td></tr>
</table>

The database configuration includes a single attribute "filePath" that may be
set to point to a JSON file containing information about how a densityEstimation decomposition
is to be stored as a file. This may be useful if the computation of a decomposition is expensive and should be stored to avoid re-computation.

### database

<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>filePath</td><td>String</td><td></td><td>Path to a JSON file containing information about how and where a decomposition is (to be) stored</td><td>densityEstimationConfig.densityEstimationType="decomposition"</td></tr>
</table>

The solver configuration controls how the SG++ solver module behaves in order to solve
systems of linear equations. solverRefineConfig applies for systems that are solved during
refinement. solverFinalConfig is used for anything else.

### solverRefineConfig

<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>solverType</td><td>String</td><td>"cg", "bicgstab", "fista"</td><td>The type of solver to be used</td><td></td></tr>
<tr><td>eps</td><td>Float</td><td>(0, inf)</td><td></td><td></td></tr>
<tr><td>maxIterations</td><td>Integer</td><td>[0, inf)</td><td>The upper boundary for solver iterations</td><td></td></tr>
<tr><td>threshold</td><td>Float</td><td>[0, inf)</td><td>An additional abort cricterium for the solver</td><td></td></tr>
</table>

### solverFinalConfig

<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>solverType</td><td>String</td><td>"cg", "bicgstab", "fista"</td><td>The type of solver to be used</td><td></td></tr>
<tr><td>eps</td><td>Float</td><td>(0, inf)</td><td></td><td></td></tr>
<tr><td>maxIterations</td><td>Integer</td><td>[0, inf)</td><td>The upper boundary for solver iterations</td><td></td></tr>
<tr><td>threshold</td><td>Float</td><td>[0, inf)</td><td>An additional abort cricterium for the solver</td><td></td></tr>
</table>

The regularization configuration controls what type and to what extend the learning
process is impacted by the regularization lambda.

### regularizationConfig

<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>regularizationType</td><td>String</td><td>"identity", "laplace", "diagonal", "lasso", "elasticNet", "groupLasso"</td><td>The method used to do regularization</td><td></td></tr>
<tr><td>lambda</td><td>Float</td><td>[0, inf)</td><td>Lambda parameter commonly used for regularization. Regulates much smoothness is enforced by the regularization.</td><td></td></tr>
<tr><td>l1Ratio</td><td>Float</td><td>[0,1]</td><td>Controls the amount of l1 regularization. A value of one corresponds to the lasso, and a value of zero to the ridge regularization</td><td>regularizationType="elasticNet"</td></tr>
<tr><td>exponentBase</td><td>Float</td><td></td><td>Currently unused</td><td>regularizationType="diagonal" (?)</td></tr>
</table>

The learner configuration applies only for classification fitting and controls
how (new) data is integrated into the model.

### learner

<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>usePrior</td><td>Boolean</td><td>true, false</td><td>Should the prior (distribution of data points into classes) be used?</td><td>fitter.type="classification"</td></tr>
<tr><td>beta</td><td>float</td><td>(0, 1)</td><td>Weight for scaling the impact of new data batches compared to old data batches</td><td>fitter.type="classification"</td></tr>
</table>

### parallelConfig

The parallelConfig deals with domain parallelization of SGDE based classification (see [here](https://mediatum.ub.tum.de/1485092)). It can be used to parallelize the SGDE learning process with ScaLAPACK.

<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>processRows</td><td>Integer</td><td>[1, inf)</td><td>optional, number of rows in the process grid.</td><td>(fitter.type="classification" OR fitter.type="densityEstimation") AND (fitter.densityEstimationConfig.densityEstimationType="decomposition") AND (fitter.densityEstimationConfig.matrixDecompositionType="chol" OR fitter.densityEstimationConfig.matrixDecompositionType="orthoadapt")</td></tr>
<tr><td>processColumns</td><td>Integer</td><td>[1, inf)</td><td>optional, number of columns in the process grid.</td><td>(fitter.type="classification" OR fitter.type="densityEstimation") AND (fitter.densityEstimationConfig.densityEstimationType="decomposition") AND (fitter.densityEstimationConfig.matrixDecompositionType="chol" OR fitter.densityEstimationConfig.matrixDecompositionType="orthoadapt")</td></tr>
<tr><td>rowBlockSize</td><td>Integer</td><td>[1, inf)</td><td>block size in the row dimension for the 2d block cyclic distribution</td><td>(fitter.type="classification" OR fitter.type="densityEstimation") AND (fitter.densityEstimationConfig.densityEstimationType="decomposition") AND (fitter.densityEstimationConfig.matrixDecompositionType="chol" OR fitter.densityEstimationConfig.matrixDecompositionType="orthoadapt")</td></tr>
<tr><td>columnBlockSize</td><td>Integer</td><td>[1, inf)</td><td>block size in the column dimension for the 2d block cyclic distribution</td><td>(fitter.type="classification" OR fitter.type="densityEstimation") AND (fitter.densityEstimationConfig.densityEstimationType="decomposition") AND (fitter.densityEstimationConfig.matrixDecompositionType="chol" OR fitter.densityEstimationConfig.matrixDecompositionType="orthoadapt")</td></tr>
</table>

If processRows or processColumns is not given (or 0 or -1), a square process grid is formed.

ScaLAPACK decomposes matrices and vectors into blocks that are cyclically mapped onto a process grid (see [here](http://www.netlib.org/scalapack/slug/node110.html#SECTION04522000000000000000), [here](http://www.netlib.org/scalapack/slug/node106.html) and [here](http://www.netlib.org/scalapack/slug/node76.html#SECTION04432000000000000000)). The parameters for the block size specify the size of the individiual blocks.

### geometryConfig

The geometryConfig structure lets you create a geometry aware sparse grid that contains much less points then a regular sparse grid. When performining e.g. image classification, the number of dimensions is too high to start with a regular sparse grid of level > 2, thus we use geometry aware sparse grids to reduce the number of grid points.

<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>dim</td><td>JSON-array</td><td>Integers for array entries</td><td>Specifies the resolution of the picture. Support: ONLY 2D Array support = Image, Default Value = std::vector<int64_t>()</td><td></td></tr>
<tr><td>stencil</td><td>String</td><td>"none", "DN"</td><td>The stencil specifies what pixel/dimension interactions are included in the a priori geometry aware grid</td><td></td></tr>
</table>


# Default and Mandatory Configuration

Many of the attributes above are optional and need not necessarily be supplied
in the JSON configuration. All (except for very few mandatory attributes) have
implicit defaults. If an invalid value - or none at all - is supplied for a
non-mandatory attribute, this built-in default is used by the pipeline.
Often it is enough to only supply a handful of attributes explicitly in the JSON
config file and rely on the defaults for all others.

In this section all attributes are listed again (this time in the JSON format)
and are explicitly set to their implicit defaults.<br>
These JSON snippets may be used to check what values are set by default or as a
template if one wants to have fine control over the pipeline configuration.

In addition the mandatory values are pointed out. If these values are omitted
it is likely that the program running the configuration will produce an error
or incorrect results. <br/>
Although the values are mandatory, they still have (often meaningless and error-inducing)
built-in defaults which, for completeness' sake, are included in the JSON snippets below.
Please make sure to always check that all mandatory attributes are provided with a value from the
JSON configuration unless you are very sure of what you are doing.

## DataSource

<b>Developer Note:</b> The implicit default values for the dataSource are set up in the struct
DataSourceConfiguration.

<b>Mandatory Values:</b> filePath

```
"dataSource": {
    "filePath": "",
    "fileType": "none",
    "compression: false,
    "numBatches": 1,
    "batchSize:" 0,
    "validationPortion": 0.3,
    "hasTargets": true,
    "dataTransformation: {
        "type": "none",
    }
    "suffling": "sequential",
    "randomSeed": -1,
    "epochs": 1,
    "readinCutoff": -1,
    "readinClasses: [ ],
    "readinColumns": [ ],
},
```

## Scorer

<b>Developer Note:</b> The implicit default value for scorer is set up in the struct
ScorerConfiguration.

<b>Mandatory Values:</b> <i>none</i>

```
"scorer": {
    "metric": "accuracy",
},
```

## Fitter

Default values for the fitter configuration are much broader and often
of limited usability. Many of the values should be manually set and adapted
to the task that the fitter has to solve.

Currently there are three types of fitter
<ul>
<li>LeastSquares</li>
<li>DensityEstimation</li>
<li>Classification (subtype of DensityEstimation)</li>
</ul>
The attribute type in the fitter configuration may be used to select one of
these three fitter types and then use a UniversalMinerFactory
to automatically construct the fitter specified in the JSON configuration.<br/>
However, it is also possible to ignore the fitter type attribute in the
JSON configuration and create the right fitter (factory) in the program
itself. This approach is the older and (as of now) the more commonly used one,
whereas the UniversalMiner\* in conjuction with the fitter type JSON attribute
is relatively new.


Here the default values for all fitter types.
Note again that DensityEstimationConfig is dependent on type=DensityEstimation.

<b>Developer Note:</b> All implicit default values are set up in FitterConfiguration::setupDefaults()
which may be (but currently are not!)
overwritten by the respective subclasses for the different fitter
types to achieve more specialization when it comes to default values.<br/>
Many attributes of the fitter configuration are initialized with values in the
struct definitions, but struct-defaults are
always overwritten by the previously mentioned setupDefaults function of FitterConfiguration or one of its subclasses.

<b>Mandatory Values:</b> type (only in conjunction with UniversalMiner\*)

```
"fitter": {

    "type": "regressionLeastSquares",

    "gridConfig": {
        "gridType": "linear",
        "dim": 0,
        "level": 3,
        "generalGridType": "regular",
        "maxDegree": 1,
        "boundaryLevel": 0,
        "fileName": "",
    },

    "adaptivityConfig": {
        "numRefinements": 0,
        "threshold": 0.0,
        "maxLevelType": false,
        "noPoints": 0,
        "percent": 1.0,
        "refinementFunctorType": "surplus",
        "refinementPeriod": 1,
        "precomputeEvaluations": true,
        "penalizeLevels": false,
        "scalingCoefficients": [],
        "errorBasedRefinement": false,
        "errorConvergenceThreshold": 0.001,
        "errorBufferSize": 3,
        "errorMinInterval": 0,
    },

    "crossValidation": {
        "enable": "false",
        "kFold": 5,
        "seed": 0,
        "shuffle": false,
        "silent": false,
        "lambda": 0.001,
        "lambdaStart": 0.001,
        "lambdaEnd": 0.001,
        "lambdaSteps": 0,
        "logScale": false,
    },

    "densityEstimationConfig": {
        "densityEstimationType": "decomposition",
        "normalize": false,
        "matrixDecompositionType": "chol",
        "iCholSweepsDecompose": "4",
        "iCholSweepsRefine": 4,
        "iCholSweepsUpdateLambda": 2,
        "iCholSweepsSolver": 2,
    },

    "database": {
        "filePath": "",
    },

    "solverRefineConfig": {
        "eps": 1e-12,
        "maxIterations": 100,
        "threshold": 1e-12,
        "solverType": "cg",
    },

    "solverFinalConfig": {
        "eps": 1e-12,
        "maxIterations": 100,
        "threshold": 1e-12,
        "solverType": "cg",
    },

    "regularizationConfig": {
        "regularizationType": "identity",
        "lambda": 0.01,
        "l1Ratio": 0.0,
        "exponentBase": 1.0,
    },

    "learner": {
        "usePrior": false,
        "beta": 1.0,
    },

    "parallelConfig": {
        "processRows": 4,
        "processColumns": 1,
        "rowBlockSize": 64,
        "columnBlockSize": 64,
    },

    "geometryConfig": {
        "dim": [28, 28],
        "stencil": "DN",
    },

},
```


# Hyperparameter Optimization

The data mining pipeline includes functionality for hyperparameter optimization
(HPO). If HPO is to be used, the config file may include an additional top-level
attribute hpp:

```
hpo_example_config.json:

{
    "dataSource": {
        ...
    },
    "scorer": {
        ...
    },
    "fitter": {
        ...
    },
    "hpo": {
        ...
    }
}
```

In order to use HPO, certain attributes described above (i.e. gridConfig.level)
change and expect different values. See below for a describtion of these changes.
In the following first the "hpo" config itself.

### hpo

<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>method</td><td>String</td><td>"harmonica", "bayesian"</td><td>The type of HPO</td><td></td></tr>
<tr><td>randomSeed</td><td>Integer</td><td>[0, inf)</td><td>Seed for random sampling within HPO</td><td></td></tr>
<tr><td>trainSize</td><td>Integer</td><td>[-1, inf)</td><td>Number of training samples to be used for HPO. -1 means all available samples are used</td><td></td></tr>
<tr><td>harmonica</td><td>Dictionary</td><td>see table below</td><td>Supply this value if harmonica-hpo should be used, otherwise omit it</td><td>method="harmonica"</td></tr>
<tr><td>bayesianOptimization</td><td>Dictionary</td><td>see table below</td><td>Supply this value if bayesianOptimization-hpo should be used, otherwise omit it</td><td>method="bayesian"</td></tr>
</table>

### harmonica

<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>lambda</td><td>Float</td><td>[0, inf)</td><td>Regularization lambda for lasso regression used by harmonica-hpo</td><td></td></tr>
<tr><td>stages</td><td>[Integer]</td><td>Each value in [0, inf)</td><td>The amount of samples to take in each stage of harmonica-hpo</td><td></td></tr>
<tr><td>constraints</td><td>[Integer]</td><td>Each value in [0, inf)</td><td>Amount of constraints to introduce after each stage of harmonica-hpo</td><td></td></tr>
</table>

### bayesianOptimization

<table>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</th><th>Comment</th><th>Depends on</th></tr>
<tr><td>nRandom</td><td>Integer</td><td>[0, inf)</td><td>Number of random samples used to warm up bayesianOptimization-hpo</td><td></td></tr>
<tr><td>nRuns</td><td>Integer</td><td>[0, inf)</td><td>Number of samples bayesianOptimization-hpo is run for</td><td></td></tr>
</table>


If HPO is used the following built-in defaults are set up by the pipeline.

<b>Developer Note:</b> All implicit default values are set up in HPOConfig::setupDefaults() except method which is not part of HPOConfig itself
and is setup in the corresponding miner factory, for instance in
MinerFactory::buildHPO().

<b>Mandatory Values:</b> <i>none</i>

```
"hpo": {
    "method": "bayesian",
    "randomSeed": 42,
    "trainSize": -1,
    "harmonica": {
        "lambda": 1.0,
        "stages": [200, 200, 200],
        "constraints": [2, 2],
    },
    "bayesianOptimization": {
        "nRandom": 10,
        "nRun": TODO,
    },
},
```

If HPO is to be used, then the structure of attributes which can be
optimized by HPO changes. Instead of providing only a single value for such
an attribute, the configuration now must include a dictionary providing
not only the usual value for that attribute but also information for the
HPO process. These additional HPO-specific attributes may for instance
include "min" and "max" attributes to restrict the search-range of the optimization.<br>
In the following all attributes which may be targeted with HPO are listed
with their <b>new</b> dictonary structure. If the attribute should not be
part of HPO, simply use the "normal", non-HPO value for the respective attribute as described above and
it will be ignore by HPO.

The particular values here are arbitrary (and are <b>not</b> default values! There are
no defaults in this context) and should be carefully chosen based
on the task.

```
"gridConfig": {
    "level": {
        "value": 3,
        "optimize": true,
        "min": 3,
        "max": 5,
    },
    "gridType": {
        "value": "linear",
        "optimize": true,
        "options": ["linear", "modlinear"],
    },
},

"adaptivityConfig": {
    "noPoints": {
        "value": 1,
        "optimize": true,
            "min": 1,
            "max": 4,
        },
    "theshold": {
        "value": -3.0,
        "optimize": true,
        "min": -5,
        "max": -1,
        "bits": 3,
        "logScale": true,
    },
},

"regularizationConfig": {
    "lambda": {
        "value": -4.0,
        "optimize": true,
        "min": -4,
        "max": -1,
        "bits": 5,
        "logScale": true,
    },
},
```