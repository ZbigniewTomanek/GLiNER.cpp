
# 👑 GLiNER.cpp: Generalist and Lightweight Named Entity Recognition for C++

GLiNER.cpp is a C++-based inference engine for running GLiNER (Generalist and Lightweight Named Entity Recognition) models. GLiNER can identify any entity type using a bidirectional transformer encoder, offering a practical alternative to traditional NER models and large language models.

<p align="center">
    <a href="https://arxiv.org/abs/2311.08526">📄 Paper</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://discord.gg/Y2yVxpSQnG">📢 Discord</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://huggingface.co/spaces/urchade/gliner_mediumv2.1">🤗 Demo</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://huggingface.co/models?library=gliner&sort=trending">🤗 Available models</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://github.com/urchade/GLiNER">🧬 Official Repo</a>
</p>

## 🌟 Key Features

- Flexible entity recognition without predefined categories
- Lightweight and fast inference;
- Ideal for running GLiNER models on CPU;

## 🚀 Getting Started
First of all, clone the repository:
```bash
git clone https://github.com/Knowledgator/GLiNER.cpp.git
```
Then you need initialize and update submodules:

```bash
cd GLiNER.cpp
git submodule update --init --recursive
```

**📦 CPU build dependencies & instructions**
 - CMake (>= 3.18)
 - [Rust and Cargo](https://www.rust-lang.org/tools/install)
 - [ONNXRuntime](https://github.com/microsoft/onnxruntime/releases) CPU version for your system
 - OpenMP 

You need to download [ONNX runtime](https://github.com/microsoft/onnxruntime/releases) for your system.

Once you downloaded it, unpack it within the same directory as GLiNER.cpp code.

For `tar.gz` files you can use the following command:
```bash
tar -xvzf onnxruntime-linux-x64-1.19.2.tgz 
```

Then create a build directory and compile the project:
```bash
cmake -D ONNXRUNTIME_ROOTDIR="/home/usr/onnxruntime-linux-x64-1.19.2" -S . -B build
cmake --build build --target all -j
```
You need to provide the ONNXRUNTIME_ROOTDIR option, which should be set to the absolute path of the chosen ONNX runtime.

To run main.cpp you need an ONNX format model and tokenizer.json. You can:

1. Search for pre-converted models on [HuggingFace](https://huggingface.co/onnx-community?search_models=gliner)
2. Convert a model yourself using the [official Python script](https://github.com/urchade/GLiNER/blob/main/convert_to_onnx.py)

### Converting to ONNX Format

Use the `convert_to_onnx.py` script with the following arguments:

- `model_path`: Location of the GLiNER model
- `save_path`: Where to save the ONNX file
- `quantize`: Set to True for IntU8 quantization (optional)

Example:

```bash
python convert_to_onnx.py --model_path /path/to/your/model --save_path /path/to/save/onnx --quantize True
```


### Example of usage:
```c++
#include <iostream>
#include <vector>
#include <string>

#include "GLiNER/gliner_config.hpp"
#include "GLiNER/processor.hpp"
#include "GLiNER/decoder.hpp"
#include "GLiNER/model.hpp"
#include "GLiNER/tokenizer_utils.hpp"

int main() {
    gliner::Config config{12, 512};  // Set your max_width and max_length
    gliner::Model model("./gliner_small-v2.1/onnx/model.onnx", "./gliner_small-v2.1/tokenizer.json", config);
    // Provide the path to the model, the path to the tokenizer, and the configuration.

    // A sample input
    std::vector<std::string> texts = {"Kyiv is the capital of Ukraine."};
    std::vector<std::string> entities = {"city", "country", "river", "person", "car"};

    auto output = model.inference(texts, entities);

    std::cout << "\nTest Model Inference:" << std::endl;
    for (size_t batch = 0; batch < output.size(); ++batch) {
        std::cout << "Batch " << batch << ":\n";
        for (const auto& span : output[batch]) {
            std::cout << "  Span: [" << span.startIdx << ", " << span.endIdx << "], "
                        << "Class: " << span.classLabel << ", "
                        << "Text: " << span.text << ", "
                        << "Prob: " << span.prob << std::endl;
        }
    }
    return 0;
}
```

## GPU
**📦Build dependencies & instruction**
 - CMake (>= 3.25)
 - [Rust and Cargo](https://www.rust-lang.org/tools/install)
 - [ONNXRuntime](https://github.com/microsoft/onnxruntime/releases) GPU version for your system
 - OpenMP
 - NVIDIA GPU
 - [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads)
 - [cuDNN](https://developer.nvidia.com/cudnn-downloads )
   
By default, the CPU is used. To use the GPU, you need to utilize the ONNX runtime with GPU support and set up cuDNN.
Follow the instructions to install cuDNN here:

https://developer.nvidia.com/cudnn-downloads

Then create a build directory and compile the project:

```bash
cmake -D ONNXRUNTIME_ROOTDIR="/home/usr/onnxruntime-linux-x64-gpu-1.19.2" -D GPU_CHECK=ON -S . -B build
cmake --build build --target inference -j
```

GPU=ON: Enables check for CUDA dependencies. If not provided, the check will be omitted.

To use GPU:

- Specify it using 'device_id':

```c++
int device_id = 0 // (CUDA:0)
gliner::Model model("./gliner_small-v2.1/onnx/model.onnx", "./gliner_small-v2.1/tokenizer.json", config, device_id);
```

OR

- Use custom environment(Ort::Env) and session options(Ort::SessionOptions) of the ONNX runtime: 

```c++
Ort::Env env = ...;
Ort::SessionOptions session_options = ...;
gliner::Model model("./gliner_small-v2.1/onnx/model.onnx", "./gliner_small-v2.1/tokenizer.json", config, env, session_options);
```

## Token-based Models

By default, the model uses a span-level configuration. To use token-level models, you need to specify the model type in the model configuration:

```c++
gliner::Config config{12, 512, gliner::TOKEN_LEVEL};  // Set your maxWidth, maxLength and modelType
gliner::Model model("./gliner-multitask-large-v0.5/onnx/model.onnx", "./gliner-multitask-large-v0.5/tokenizer.json", config);
```

## 🌟 Use Cases

GLiNER.cpp offers versatile entity recognition capabilities across various domains:

1. **Enhanced Search Query Understanding**
2. **Real-time PII Detection**
3. **Intelligent Document Parsing**
4. **Content Summarization and Insight Extraction**
5. **Automated Content Tagging and Categorization**
...

## 🔧 Areas for Improvement
- [x] Add support of token-level GLiNER models;
- [ ] Further optimize inference speed;
- [ ] Implement bi-encoder GLiNER architecture for better scalability;
- [ ] Enable model training capabilities;
- [ ] Provide more usage examples.


## 🙏 Acknowledgements

- [GLiNER original authors](https://github.com/urchade/GLiNER)
- [ONNX Runtime Web](https://github.com/microsoft/onnxruntime)

## 📞 Support

For questions and support, please join our [Discord community](https://discord.gg/ApZvyNZU) or open an issue on GitHub.
