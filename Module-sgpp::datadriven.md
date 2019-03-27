Data mining and machine learning.

# Overview
Functionality includes
- operations depending on datasets
- data mining support (data mining pipeline)
- specialized regularization
- vectorization of evaluation of multiple on x86

# Data mining Pipeline

The data mining pipeline is a submodule of `sgpp::datadriven` intended for
out-of-the-box application of SG++ machine learning methods.
By using the pipeline many algorithms of the
datadriven module can be applied with minimal setup in C++. All boilerplate code
is hidden from the user and configuration is done via JSON config files
(see below).

The datamining pipeline is configured via JSON config files. Please refer to
the [configuration page](https://github.com/SGpp/SGpp/wiki/?)
for documentation of the configuration format, attributes and default values.

# Doxygen convert Test

## Code Block Test

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

## Table Test

<table>
<caption>dataTransformation</caption>
<tr><th>Attribute Name</th><th>Attribute Type</th><th>Valid value range</tr><th>Comment</th><th>Depends on</th></tr>
<tr><td>type</td><td>String</td><td>"rosenblatt", "none"</td><td>The type of data transformation. Currently only rosenblatt transformation is available</td><td></td></tr>
</table>
