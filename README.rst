Using Mojo Pipeline with Spark/Sparkling Water
==============================================

Mojo pipeline artifacts can be used in Spark to deploy predictions in parallel using the Sparkling Water API.
The user needs to have Spark cluster with Sparkling Water JAR file passed to Spark. In case of PySparkling, the PySparkling zip file.

The H2OContext does not have to be created in case we want to run only predictions on mojos using Spark as they are written to be independent
on the H2O run-time.

The following sections show how to load and run predictions on Mojo Pipeline in Spark using Scala and Python API

Preparing Environment
---------------------

Both, PySparkling and Sparkling Water needs to be started with some extra configuration to enable Mojo Pipeline

We need to pass the path to the H2O Driverless AI license to ``--jars`` Spark's argument. Additionally, we need to add to the same ``--jars`` 
configuration path to the Mojo Pipeline implementation JAR file. This file is propriatory and is not part of the resulting Sparkling Water assembly
JAR file.

	Note: In Local Spark mode, please use ``--driver-class-path`` to specify path to the license and Mojo Pipeline JAR file.

PySparkling
-----------

First, let's start Spark with all the required dependencies



..code:: bash

	./bin/pyspark --jars license.sig,mojo-pipeline-impl.jar --py-files pysparkling.zip

We can see that we passed the license and mojo pipeline implementation library to the ``--jars`` argument and also specified path
to the PySparkling python library. At this point, we should have available PySpark interactive terminal where we can try our predictions. In case you would like
to productionalize this, you can use the same configuration except instead of using ``./bin/pyspark`` you would use ``./bin/spark-submit`` to
submit your job to a cluster.

..code:: python
	
	# First, specify the dependency
	from pysparkling.ml import H2OMOJOPipelineModel

..code:: python

	# Load the pipeline
	mojo = H2OMOJOPipelineModel.create_from_mojo("file:///path/to/the/pipeline.mojo")
	# This option ensures that the output columns are named properly. If you want to use old behaviour
	# when all output columns were stored inside and array, don't specify this configuration option
	# or set it to False. We however strongly encourage users to use
	mojo.set_named_mojo_output_columns(True)

..code:: python

	# Load the data as Spark's Data Frame
	data_frame = self._spark.read.csv("file:///path/to/the/data.csv", header=True)

..code:: python

	# Run the predictions. The predictions contain all the original columns plus the predictions added as new columns
	predictions = mojo.predict(data_frame)
	# We can easily get the predictions for desired column using the helper function as
	predictions.select(mojo.select_prediction_udf("AGE")).collect()
	
Sparkling Water
---------------