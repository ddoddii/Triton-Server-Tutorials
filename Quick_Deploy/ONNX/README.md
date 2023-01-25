# Deploying an ONNX Model

This README showcases how to deploy a simple ResNet model on Triton Inference Server.

## Step 1: Set Up Triton Inference Server

To use Triton, we need to build a model repository. The structure of the repository as follows:
```
model_repository
|
+-- resnet
    |
    +-- config.pbtxt
    +-- 1
        |
        +-- model.onnx
```

The `config.pbtxt` configuration file is optional. The configuration file is autogenerated by Triton Inference Server if the user doesn't provide it. If you are new to Triton, it is highly recommended to [review Part 1](../../Conceptual_Guide/Part_1-model_deployment/README.md) of the conceptual guide.
```
mkdir -p model_repository/densenet_onnx/1
wget -O model_repository/densenet_onnx/1/model.onnx \
     https://contentmamluswest001.blob.core.windows.net/content/14b2744cf8d6418c87ffddc3f3127242/9502630827244d60a1214f250e3bbca7/08aed7327d694b8dbaee2c97b8d0fcba/densenet121-1.2.onnx


docker run --gpus all --rm -p 8000:8000 -p 8001:8001 -p 8002:8002 -v ${PWD}/model_repository:/models nvcr.io/nvidia/tritonserver:<xx.yy>-py3 tritonserver --model-repository=/models
```

## Step 3: Using a Triton Client to Query the Server

Install dependencies & download an example image to test inference.

```
docker run -it --rm --net=host -v ${PWD}:/workspace/ nvcr.io/nvidia/tritonserver:<yy.mm>-py3-sdk bash
pip install torchvision

wget  -O img1.jpg "https://www.hakaimagazine.com/wp-content/uploads/header-gulf-birds.jpg"
```
Building a client requires three basic points. Firstly, we setup a connection with the Triton Inference Server.
```
client = httpclient.InferenceServerClient(url="localhost:8000")
```
Secondly, we specify the names of the input and output layer(s) of our model as well as describe the shape and datatype of the expected input.
```
inputs = httpclient.InferInput("data_0", transformed_img.shape, datatype="FP32")
inputs.set_data_from_numpy(transformed_img, binary_data=True)

outputs = httpclient.InferRequestedOutput("fc6_1", binary_data=True, class_count=1000)
```
Lastly, we send an inference request to the Triton Inference Server.
```
# Querying the server
results = client.infer(model_name="densenet_onnx", inputs=[inputs], outputs=[outputs])
inference_output = results.as_numpy('fc6_1').astype(str)

print(np.squeeze(inference_output)[:5])
```
The output of the same should look like below:
```
['11.549026:92' '11.232335:14' '7.528014:95' '6.923391:17' '6.576575:88']
```
The output format here is `<confidence_score>:<classification_index>`. To learn how to map these to the label names and more, refer to our [documentation](https://github.com/triton-inference-server/server/blob/main/docs/protocol/extension_classification.md). The client code above is available in `client.py`. 