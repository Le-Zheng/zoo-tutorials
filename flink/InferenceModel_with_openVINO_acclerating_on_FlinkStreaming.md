# Analytics-Zoo InferenceModel with openVINO acclerating  on Flink Streaming 

`model-inference-flink` is the model inference in batch and streaming with flink. This is the example of batch and streaming with Flink and Resnet50 model, as well as using Analytics-Zoo InferenceModel to acclerate prediction. See [here](https://github.com/glorysdj/analytics-zoo/blob/imflink2/apps/model-inference-examples/model-inference-flink/src/main/scala/com/intel/analytics/zoo/apps/model/inference/flink/ImageClassificationStreaming.scala) for the whole program.

## Getting started Aalytics-Zoo InferenceModel
Define a class extended analytics-zoo `InferenceModel`. It allows to pass modelType, modelBytes, inputShape, ifReverseInputChannels, meanValues and scale to convert to openVINO model. And load the whole parameters using `doLoadTF` method.
This is the sample of defining a `Resnet50InferenceModel` class. See [here](https://github.com/glorysdj/analytics-zoo/blob/imflink2/apps/model-inference-examples/model-inference-flink/src/main/scala/com/intel/analytics/zoo/apps/model/inference/flink/Resnet50InferenceModel.scala) for the whole program.

```
package com.intel.analytics.zoo.apps.model.inference.flink
import java.nio.channels.Channels
import com.intel.analytics.zoo.pipeline.inference.InferenceModel
class Resnet50InferenceModel(var concurrentNum: Int = 1, modelType: String, modelBytes: Array[Byte], inputShape: Array[Int], ifReverseInputChannels: Boolean, meanValues: Array[Float], scale: Float) extends InferenceModel(concurrentNum) with Serializable {
  doLoadTF(null, modelType, written.getAbsolutePath, inputShape, ifReverseInputChannels, meanValues, scale)
  println(this)
}
```
 
## Getting started Flink program

### Obtain an execution environment 
The `StreamExecutionEnvironment` is the context in which a streaming program is executed. `getExecutionEnvironment` is the typical function creating an environment to execute your program when the program is invoked on your local machine or a cluster.
```
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
```
### Create and transform DataStreams
`StreamExecutionEnvironment` supports creating a DataStream from a collection using `fromCollection()` method. 
```
import org.apache.flink.streaming.api.datastream.DataStreamUtils
import com.intel.analytics.zoo.pipeline.inference.JTensor
import java.util.{List => JList}
import java.util.Arrays
val dataStream: DataStream[Array[Float]] =  env.fromCollection(inputs)
val tensorStream: DataStream[JList[JList[JTensor]]] = dataStream.map(value => {
val input = new JTensor(value, Array(1, 224, 224, 3))
val data = Arrays.asList(input)
List(data).asJava
})
```
### Specifying Transformation Functions
Define a class extends `RichMapFunction`. Three main methods of rich function in this example are open, close and map. `open()` is initialization method. `close()` is called after the last call to the the main working methods. `map()` is the user-defined function, mapping an element from the input data set and to one exact element, ie, `JList[JList[JTensor]]`.
```
import org.apache.flink.api.common.functions.RichMapFunction
class ModelPredictionMapFunction(modelType: String, modelBytes: Array[Byte], inputShape: Array[Int], ifReverseInputChannels: Boolean, meanValues: Array[Float], scale: Float) extends RichMapFunction[JList[JList[JTensor]], JList[JList[JTensor]]] {
  var resnet50InferenceModel: Resnet50InferenceModel = _

  override def open(parameters: Configuration): Unit = {
    resnet50InferenceModel = new Resnet50InferenceModel(1, modelType, modelBytes, inputShape, ifReverseInputChannels, meanValues, scale)
  }

  override def close(): Unit = {
    resnet50InferenceModel.release()
  }

  override def map(in: JList[JList[JTensor]]): JList[JList[JTensor]] = {
    resnet50InferenceModel.doPredict(in)
  }
``` 
Pass the `RichMapFunctionn` function to a `map` transformation.
```
val resultStream = tensorStream.map(new ModelPredictionMapFunction(modelType, modelBytes, inputShape, ifReverseInputChannels, meanValues, scale))
``` 
### Trigger the program execution 
The program is actually executed when calling `execute()` on the `StreamExecutionEnvironment`. Whether the program is executed locally or submitted on a cluster depends on the type of execution environment.
```
env.execute()
```
### Transform collections of data
Create an iterator to iterate over the elements of the DataStream.
```
import org.apache.flink.streaming.api.datastream.DataStreamUtils
val results = DataStreamUtils.collect(resultStream.javaStream).asScala
```
## How to run the example
### Requirements
* JDK 1.8
* Flink 1.8.1
* scala 2.11/2.12
* Python 3.x

### Environment
Install dependencies for each flink node.
```
sudo apt install python3-pip
pip3 install numpy
pip3 install networkx
pip3 install tensorflow
```
### Start and stop Flink
you may start a flink cluster if there is no runing one:
```
./bin/start-cluster.sh
```
Check the Dispatcher's web frontend at http://localhost:8081 and make sure everything is up and running.
To stop Flink when you're done type:
```
./bin/stop-cluster.sh
```

### Run the Example
* Run `export FLINK_HOME=the root directory of flink`.
* Run `export ANALYTICS_ZOO_HOME=the folder of Analytics Zoo project`.
* Download [resnet_v1_50 model](http://download.tensorflow.org/models/resnet_v1_50_2016_08_28.tar.gz). Run `export MODEL_PATH=path to the downloaded model`.
* Go to the root directory of model-inference-flink and execute the `mvn clean package` command, which prepares the jar file for model-inference-flink.
* Edit flink-conf.yaml to set heap size or the number of task slots as you need, ie,  `jobmanager.heap.size: 10g`
* Run the follwing command with arguments to submit the Flink program. Change parameter settings as you need.

```bash
${FLINK_HOME}/bin/flink run \
    -m localhost:8181 -p 2 \
    -c com.intel.analytics.zoo.apps.model.inference.flink.ImageClassificationStreaming  \
    ${ANALYTICS_ZOO_HOME}/apps/model-inference-examples/model-inference-flink/target/model-inference-flink-0.1.0-SNAPSHOT-jar-with-dependencies.jar  \
    --modelType resnet_v1_50 --checkpointPathcheckpointPath ${MODEL_PATH}  \
    --inputShape "1,224,224,3" --ifReverseInputChannels true --meanValues "123.68,116.78,103.94" --scale 1
```

