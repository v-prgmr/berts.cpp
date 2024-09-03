# berts.cpp on Android

[ggml](https://github.com/ggerganov/ggml) inference of bert family models (bert, distilbert, roberta ...), classification & seq2seq and more.
High quality bert inference in pure C++.

## Motivation for BERT model on android 
### Why use berts.cpp instead of llama.cpp (or) bert.cpp?
 
* [llama.cpp](https://github.com/ggerganov/llama.cpp) and [bert.cpp](https://github.com/skeskinen/bert.cpp/) does not 
 have support for BertForSequenceClassification architecture. 
* Even though [bert.cpp](https://github.com/skeskinen/bert.cpp/) is integrated into llama.cpp →
https://github.com/ggerganov/llama.cpp/pull/5423, they only support BertEmbedding models like all-MiniLM-L6-v2. 
* And [bert.cpp](https://github.com/skeskinen/bert.cpp/) inference code needs to be refactored for making it work 
with SequenceClassification model architecture like mentioned by the author here in https://github.com/skeskinen/bert.cpp/issues/11 
which seems to be tedious and also the OP claims that they have tried using available operator set from GGML library and experiences 
 erroneous/ unexpected output.

Thanks to [yilong2001/berts.cpp](https://github.com/yilong2001/berts.cpp) which is an implementation that does sequence 
classification in pure C++, modify the CMakeLists.txt with missing compiler options for aarch64, we can build the project 
for android armv8-a using [Android NDK r25](https://developer.android.com/ndk/downloads/revision_history). 

To run bertsbase.cpp on an embedded android platform, follow the instructions under `Build for Android` section.

## Description
The main goal of `berts.cpp` is to run the BERT model with simple binary on CPU

* Plain C/C++ implementation without dependencies
* Inherit support for various architectures from ggml (x86 with AVX2, ARM, etc.)
* Choose model size from 32/16 bits per model weigth
* Simple main for using
* CPP rest server
* Benchmarks to validate correctness and speed of inference

## Limitations & TODO
* Add classification string as an argument that can be passed from commandline
* Implement argmax of inference output to find the index that corresponds to the class
* bert seq2seq
* bard 
* xlnet
* gpt2
* ...

## Usage

### Checkout the ggml submodule
```sh
git submodule update --init --recursive
```
### Download models
Bert sequence classification model provided as a example. 
You can download with the following cmd or directly from huggingface [https://huggingface.co/yilong2001/bert_cls_example].

```sh
pip3 install -r requirements.txt
python3 models/download-ggml.py download bert-base-uncased f32
```

### Install External Library
To build the library or binary, need install external library
```
# utf8proc
# oatpp

# after intall oatpp, need set lib and include path (set actual path in your env):

# export LIBRARY_PATH=/usr/local/lib/oatpp-1.3.0:$LIBRARY_PATH
# export LD_LIBRARY_PATH=/usr/local/lib/oatpp-1.3.0:$LD_LIBRARY_PATH
# export CPLUS_INCLUDE_PATH=/usr/local/include/oatpp-1.3.0/oatpp:$CPLUS_INCLUDE_PATH

```
### Build for Android

* You can use your own custom finetuned BERT model for classifying a sentence.
* First convert your finetuned BERT model following the instructions under `Converting models to ggml format` section.
* Replace the string in line 41 of bert-main.cpp with your string that has to be classified. (This can be set as a
cmd line argument in the future) 
* To compile bert-main.cpp for android,

```
mkdir build_android
cd build_android
export NDK=~/Android/Sdk/ndk/25.1.8937393 # locate your local NDK installation directory
cmake -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-23 -DCMAKE_C_FLAGS=-march=armv8.4a+dotprod .. -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release ..
make
```
* This should create the `bert-main` inside the build folder.
* Now to classify using your BERT model, copy the binary file and the BERT model to your android device and run

```commandline
bert-main -m your_BERT_model.bin
```
* This returns a tensor which corresponds to the probabilities of all the classes that your model was trained to classify.
* Taking argmax of this tensor should return the indices of your class. (could be implemented in the future)

### Build
To build the dynamic library for usage from e.g. Golang:
```sh
mkdir build
cd build
cmake .. -DBUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release
make
cd ..
```

To build the native binaries, like the example server, with static libraries, run:
```sh
mkdir build
cd build
cmake .. -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release
make
cd ..
```


### Run sample main
```sh
# ./build/bin/bert-main -m models/bert-base-uncased/ggml-model-f32.bin

# bertencoder_load_from_file: loading model from 'models/bert-base-uncased/ggml-model-f32.bin' - please wait ...
# bertencoder_load_from_file: n_vocab = 30522
# bertencoder_load_from_file: max_position_embeddings   = 512
# bertencoder_load_from_file: intermediate_size  = 3072
# bertencoder_load_from_file: num_attention_heads  = 12
# bertencoder_load_from_file: num_hidden_layers  = 12
# bertencoder_load_from_file: pad_token_id  = 0
# bertencoder_load_from_file: n_embd  = 768
# bertencoder_load_from_file: f16     = 0
# bertencoder_load_from_file: ggml ctx size = 417.73 MB
# bertencoder_load_from_file: ......................... done
# bertencoder_load_from_file: model size =   417.65 MB / num tensors = 201
# bertencoder_load_from_file: mem_per_token 0 KB, mem_per_input 0 MB
# main: number of tokens in prompt = 7


# main:    load time =   156.61 ms
# main:    eval time =    32.76 ms / 4.68 ms per token
# main:    total time =   189.38 ms

```

### Start rest server
```sh
./build/bin/bert-rest -m models/bert-base-uncased/ggml-model-f32.bin --port 8090

# bertencoder_load_from_file: loading model from 'models/bert-base-uncased/ggml-model-f32.bin' - please wait ...
# bertencoder_load_from_file: n_vocab = 30522
# bertencoder_load_from_file: max_position_embeddings   = 512
# bertencoder_load_from_file: intermediate_size  = 3072
# bertencoder_load_from_file: num_attention_heads  = 12
# bertencoder_load_from_file: num_hidden_layers  = 12
# bertencoder_load_from_file: pad_token_id  = 0
# bertencoder_load_from_file: n_embd  = 768
# bertencoder_load_from_file: f16     = 0
# bertencoder_load_from_file: ggml ctx size = 417.73 MB
# bertencoder_load_from_file: ......................... done
# bertencoder_load_from_file: model size =   417.65 MB / num tensors = 201
# bertencoder_load_from_file: mem_per_token 0 KB, mem_per_input 0 MB

#  I |2023-11-05 00:05:29 1699113929846361| MyApp:Server running on port 8090
```


### Converting models to ggml format
Converting models is similar to llama.cpp. Use models/bert-classify-to-ggml.py to make hf models into either f32 or f16 ggml models. 

```sh
cd models
# Clone a model from hf
git clone [https://huggingface.co/yilong2001/bert_cls_example]
# Run conversions to 4 ggml formats (f32, f16)
sh run_conversions.sh bert-base-uncased 0
```

