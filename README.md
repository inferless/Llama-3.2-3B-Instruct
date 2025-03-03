# Tutorial - Deploy Llama-3.2-3B-Instruct using Inferless
[Llama-3.2-3B-Instruct](https://huggingface.co/meta-llama/Llama-3.2-3B-Instruct) is Meta’s streamlined, 3B small language model fine-tuned for instruction following and conversational applications. This model is optimized for multilingual dialogue, retrieval, summarization, and prompt rewriting, delivering impressive performance despite its modest size. Its efficient, auto-regressive transformer architecture with innovations like Grouped-Query Attention ensures low latency and ease of deployment, making it ideal for local inference on a wide range of hardware, from edge devices to personal laptops. Whether used as a base for further fine-tuning or as a ready-to-use conversational agent, Llama-3.2-3B-Instruct offers a compelling balance of speed, capability, and accessibility. 

## TL;DR:
- Deployment of Llama-3.2-3B-Instruct model using [vLLM](https://github.com/vllm-project/vllm/).
- Dependencies defined in `inferless-runtime-config.yaml`.
- GitHub/GitLab template creation with `app.py`, `inferless-runtime-config.yaml` and `inferless.yaml`.
- Model class in `app.py` with `initialize`, `infer`, and `finalize` functions.
- Custom runtime creation with necessary system and Python packages.
- Recommended GPU: NVIDIA A100.
- Custom runtime selection in advanced configuration.
- Final review and deployment on the Inferless platform.

### Fork the Repository
Get started by forking the repository. You can do this by clicking on the fork button in the top right corner of the repository page.

This will create a copy of the repository in your own GitHub account, allowing you to make changes and customize it according to your needs.

### Create a Custom Runtime in Inferless
To access the custom runtime window in Inferless, simply navigate to the sidebar and click on the Create new Runtime button. A pop-up will appear.

Next, provide a suitable name for your custom runtime and proceed by uploading the inferless-runtime.yaml file given above. Finally, ensure you save your changes by clicking on the save button.

### Add Your Hugging Face Access Token
Go into the `inferless.yaml` and replace `<hugging_face_token>` with your hugging face access token. Make sure to check the repo is private to protect your hugging face token.

### Import the Model in Inferless
Log in to your inferless account, select the workspace you want the model to be imported into and click the Add Model button.

Select the PyTorch as framework and choose **Repo(custom code)** as your model source and select your provider, and use the forked repo URL as the **Model URL**.

Enter all the required details to Import your model. Refer [this link](https://docs.inferless.com/integrations/git-custom-code/git--custom-code) for more information on model import.

---
## Curl Command
Following is an example of the curl command you can use to make inference. You can find the exact curl command in the Model's API page in Inferless.
```bash
curl --location '<your_inference_url>' \
    --header 'Content-Type: application/json' \
    --header 'Authorization: Bearer <your_api_key>' \
    --data '{
      "inputs": [
        {
          "name": "prompt",
          "shape": [1],
          "data": ["What is deep learning?"],
          "datatype": "BYTES"
        },
        {
          "name": "temperature",
          "optional": true,
          "shape": [1],
          "data": [0.7],
          "datatype": "FP32"
        },
        {
          "name": "top_p",
          "optional": true,
          "shape": [1],
          "data": [0.1],
          "datatype": "FP32"
        },
        {
          "name": "repetition_penalty",
          "optional": true,
          "shape": [1],
          "data": [1.18],
          "datatype": "FP32"
        },
        {
          "name": "max_tokens",
          "optional": true,
          "shape": [1],
          "data": [512],
          "datatype": "INT16"
        },
        {
          "name": "top_k",
          "optional": true,
          "shape": [1],
          "data": [40],
          "datatype": "INT8"
        }
      ]
    }'
```

---
## Customizing the Code
Open the `app.py` file. This contains the main code for inference. The `InferlessPythonModel` has three main functions, initialize, infer and finalize.

**Initialize** -  This function is executed during the cold start and is used to initialize the model. If you have any custom configurations or settings that need to be applied during the initialization, make sure to add them in this function.

**Infer** - This function is where the inference happens. The infer function leverages both RequestObjects and ResponseObjects to handle inputs and outputs in a structured and maintainable way.
- RequestObjects: Defines the input schema, validating and parsing the input data.
- ResponseObjects: Encapsulates the output data, ensuring consistent and structured API responses.

```python
  def infer(self, request: RequestObjects) -> ResponseObjects:
    sampling_params = SamplingParams(temperature=request.temperature,top_p=request.top_p,repetition_penalty=request.repetition_penalty,
                     top_k=request.top_k,max_tokens=request.max_tokens)
    input_text = self.tokenizer.apply_chat_template([{"role": "user", "content": request.prompt}], tokenize=False)
    result = self.llm.generate(input_text, sampling_params)
    result_output = [output.outputs[0].text for output in result]

    generateObject = ResponseObjects(generated_text = result_output[0])        
    return generateObject
```

**Finalize** - This function is used to perform any cleanup activity for example you can unload the model from the gpu by setting to `None`.
```python
def finalize(self):
    self.llm = None
```


For more information refer to the [Inferless docs](https://docs.inferless.com/).
