# poll-classifier

![Tests](https://github.com/roseinstitutecmc/poll-classifier/actions/workflows/python-tests.yml/badge.svg)

## Overview
This project classifies free response poll issues using a chat language model. The input is a CSV file where each row is a response. The language model outputs a list of issues coded following the provided codebook.

## Setup
1. Clone the repository.
2. Install the required dependencies.

python >= 3.10

pip3 install torch torchvision

pip install llama-cpp-python
```
pip install llama-cpp-python \
  --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/metal
```
(alt is step 4 of https://llama-cpp-python.readthedocs.io/en/latest/install/macos/)

downloading model:
from https://huggingface.co/bartowski/Llama-3.2-3B-Instruct-GGUF

pip install -U "huggingface_hub[cli]"

huggingface-cli download bartowski/Llama-3.2-3B-Instruct-GGUF --include "Llama-3.2-3B-Instruct-Q4_0.gguf" --local-dir ./models
(might need the tokenizer as well?)

(needs to be Q4_0.gguf according to https://llama-cpp-python.readthedocs.io/en/latest/install/macos/. This is a quantized model.)


pip install lmql[hf]


lmql serve-model llama.cpp:<PATH TO WEIGHTS>.gguf
lmql serve-model "llama.cpp:../../../../models/Llama-3.2-3B-Instruct-Q4_0.gguf"
(This funny path is related to https://github.com/eth-sri/lmql/issues/344)


This isn't working.

```
❯ lmql serve-model "llama.cpp:../../../../models/Llama-3.2-3B-Instruct-Q4_0.gguf"

[Serving LMTP endpoint on ws://localhost:8080/]
[Loading llama.cpp model from llama.cpp:../../../../models/Llama-3.2-3B-Instruct-Q4_0.gguf  with  {} ]
```

Then this when I try to run the test:
```
  File "/Users/nolan/Documents/GitHub/poll-classifier/venv/lib/python3.10/site-packages/llama_cpp/llama.py", line 703, in apply_func
    for logit_processor in logits_processor:
TypeError: 'BatchLogitsProcessor' object is not iterable
Exception ignored on calling ctypes callback function: <function CustomSampler.__init__.<locals>.apply_wrapper at 0x115b21ab0>
Traceback (most recent call last):
  File "/Users/nolan/Documents/GitHub/poll-classifier/venv/lib/python3.10/site-packages/llama_cpp/_internals.py", line 725, in apply_wrapper
    self.apply_func(cur_p)
```

This is what I get on the client side:
```
python test_lmql.py
/Users/nolan/Documents/GitHub/poll-classifier/venv/lib/python3.10/site-packages/lmql/api/llm.py:220: UserWarning: File tokenizer.model not present in the same folder as the model weights. Using default 'huggyllama/llama-7b' tokenizer for all llama.cpp models. To change this, set the 'tokenizer' argument of your lmql.model(...) object.
  warnings.warn("File tokenizer.model not present in the same folder as the model weights. Using default '{}' tokenizer for all llama.cpp models. To change this, set the 'tokenizer' argument of your lmql.model(...) object.".format("huggyllama/llama-7b", UserWarning))
You are using the default legacy behaviour of the <class 'transformers.models.llama.tokenization_llama_fast.LlamaTokenizerFast'>. This is expected, and simply means that the `legacy` (previous) behavior will be used so nothing changes for you. If you want to use the new behaviour, set `legacy=False`. This should only be set if you understand what it means, and thoroughly read the reason why this was added as explained in https://github.com/huggingface/transformers/pull/24565 - if you loaded a llama tokenizer from a GGUF file you can ignore this message.
```


Going to try the alt step 4 of https://llama-cpp-python.readthedocs.io/en/latest/install/macos/

pip uninstall llama-cpp-python -y
CMAKE_ARGS="-DGGML_METAL=on" pip install -U llama-cpp-python --no-cache-dir
pip install 'llama-cpp-python[server]'

export MODEL=models/Llama-3.2-3B-Instruct-Q4_0.gguf
python3 -m llama_cpp.server --model $MODEL  --n_gpu_layers 1

(this looks much better, but it looks like the server needs to be run by lmql because they are doing their own thing.)


Still getting issues, even when trying to run without server.

```
❯ python test_lmql.py
/Users/nolan/Documents/GitHub/poll-classifier/venv/lib/python3.10/site-packages/lmql/models/lmtp/lmtp_dcinprocess.py:36: UserWarning: By default LMQL uses the 'huggyllama/llama-7b' tokenizer for all llama.cpp models. To change this, set the 'tokenizer' argument of your lmql.model(...) object.
  warnings.warn("By default LMQL uses the '{}' tokenizer for all llama.cpp models. To change this, set the 'tokenizer' argument of your lmql.model(...) object.".format("huggyllama/llama-7b", UserWarning))
You are using the default legacy behaviour of the <class 'transformers.models.llama.tokenization_llama_fast.LlamaTokenizerFast'>. This is expected, and simply means that the `legacy` (previous) behavior will be used so nothing changes for you. If you want to use the new behaviour, set `legacy=False`. This should only be set if you understand what it means, and thoroughly read the reason why this was added as explained in https://github.com/huggingface/transformers/pull/24565 - if you loaded a llama tokenizer from a GGUF file you can ignore this message.
[Loading llama.cpp model from llama.cpp:models/Llama-3.2-3B-Instruct-Q4_0.gguf  with  {} ]
Exception ignored on calling ctypes callback function: <function CustomSampler.__init__.<locals>.apply_wrapper at 0x10dc55fc0>
Traceback (most recent call last):
  File "/Users/nolan/Documents/GitHub/poll-classifier/venv/lib/python3.10/site-packages/llama_cpp/_internals.py", line 725, in apply_wrapper
    self.apply_func(cur_p)
  File "/Users/nolan/Documents/GitHub/poll-classifier/venv/lib/python3.10/site-packages/llama_cpp/llama.py", line 703, in apply_func
    for logit_processor in logits_processor:
TypeError: 'BatchLogitsProcessor' object is not iterable
Exception ignored on calling ctypes callback function: <function CustomSampler.__init__.<locals>.apply_wrapper at 0x10dc55fc0>
```


CURRENT STATE:
BLOCKED ON GETTING LMQL TO WORK WITH LLAMA CPP.
LLAMA CPP WORKS ON ITS OWN.


lmql.model defined at: https://github.com/eth-sri/lmql/blob/3db7201403da4aebf092052d2e19ad7454158dd7/src/lmql/api/llm.py#L318
Then this alias from_descriptor. Maybe I'm having issue with tokenizer?
https://github.com/eth-sri/lmql/blob/3db7201403da4aebf092052d2e19ad7454158dd7/src/lmql/api/llm.py#L216


Did something and am now getting output along with errors, but its garbeled. Try getting tokenizer.

Get access to Llama-3.2-3B-Instruct. Login. to hf cli. Or drag and drop.

huggingface-cli download meta-llama/Llama-3.2-3B-Instruct --include "original/tokenizer.model" --local-dir ./models

When passing tokenizer, getting new error!
```
  File "/Users/nolan/Documents/GitHub/poll-classifier/venv/lib/python3.10/site-packages/lmql/runtime/tokenizer.py", line 83, in tokenizer_impl
    tokenizer_not_found_error(self.model_identifier)
  File "/Users/nolan/Documents/GitHub/poll-classifier/venv/lib/python3.10/site-packages/lmql/runtime/tokenizer.py", line 366, in tokenizer_not_found_error
    raise TokenizerNotAvailableError("Failed to locate a suitable tokenizer implementation for '{}' (Make sure your current environment provides a tokenizer backend like 'transformers', 'tiktoken' or 'llama.cpp' for this model)".format(model_identifier))
lmql.runtime.tokenizer.TokenizerNotAvailableError: Failed to locate a suitable tokenizer implementation for '/Users/nolan/Documents/GitHub/poll-classifier/models/tokenizer.model' (Make sure your current environment provides a tokenizer backend like 'transformers', 'tiktoken' or 'llama.cpp' for this model)
```

Next, figure that out.




Ok, now I know that this works, but has a looping error/warning message I don't understand.

Server code. Running with `--n_gpu_layers -1` enables GPU acceleration and is faster than CPU. Its not as fast as llama.cpp.
```
lmql serve-model "llama.cpp:../../../../models/Llama-3.2-3B-Instruct-Q4_0.gguf" --n_ctx 1024 --n_gpu_layers -1
```
Client code. Note it must be run from within a function:
```
@lmql.query(
    model=lmql.model(
        "llama.cpp:../../../../models/Llama-3.2-3B-Instruct-Q4_0.gguf", 
        tokenizer="meta-llama/Llama-3.2-3B-Instruct",
    )
)
```



You should review the whole installation instructions on the llama-cpp-python repo for your specific system. https://github.com/abetlen/llama-cpp-python?tab=readme-ov-file#installation
The defualt is a MacOS M1 Max.


References used while installing and troubleshooting, note all have varrying degrees of correctness and usefulness.

https://lmql.ai/docs/lib/generations.html
https://lmql.ai/docs/models/llama.cpp.html
https://github.com/eth-sri/lmql/blob/3db7201403da4aebf092052d2e19ad7454158dd7/src/lmql/models/lmtp/backends/llama_cpp_model.py
https://github.com/eth-sri/lmql/blob/main/src/lmql/api/llm.py#L68
https://github.com/eth-sri/lmql/blob/main/src/lmql/models/lmtp/README.md
https://github.com/eth-sri/lmql/blob/3db7201403da4aebf092052d2e19ad7454158dd7/src/lmql/models/lmtp/lmtp_serve.py#L100
https://llama-cpp-python.readthedocs.io/en/latest/install/macos/
https://github.com/abetlen/llama-cpp-python?tab=readme-ov-file#installation

https://github.com/eth-sri/lmql/issues/350
https://github.com/eth-sri/lmql/issues/274




This library follows the ____ system for generating outputs.


There is also an ability to test the system against ground truth data.

You can select a random sample of cases to test the system, that randomness of the selecition is controlled by the `seed` parameter.


The steps for each classification for each case are:

1. Encode the items to be classified into a string following a given template.
2. Pass the string to the language model.
3. 

TODO:

Adding a Chain of Thought step before final answer might improve results.



data needs to be a list of tuples with first element being an identifer key and second element being a string of the text to be classified.
```
data = [
    ("id1", "text1"),
    ("id2", "text2"),
    ("id3", "text3"),
]

ds = Dataset(data)
```

Dataset objects have several methods.

hash method returns a unique hash for the dataset.

```
ds.hash()
# "a1b2c3d4e5f6g7h8i9j0"
```

sample method returns a random sample of the dataset.
n is the number of samples to return.
seed is the random seed to use for the sample.

```
ds.sample(n=3, seed=42)
# [("id2", "text2"), ("id3", "text3"), ("id1", "text1")]
```

Model objects are configured as a predictor. You can pass prompts, valid labels, language model objects, and other parameters to the constructor.

```
m = Model(
    prompt="Review: {review}",
    valid_labels=["A", "B", "C"],
    model=lmql.model("llama.cpp:../../../../models/Llama-3.2-3B-Instruct-Q4_0.gguf"),
)
```

Model objects have a predict method that takes a dataset as input and returns a list of predictions. Some models may return return a list of predictions per item in the dataset.

```
m.predict(ds)
# [("id1", "A"), ("id2", "B"), ("id3", ["A", "C"])]
```

You may indicate that the predict method should also return the confidence scores for each prediction.

```
m.predict(ds, return_confidences=True)
# [("id1", "A", 0.9), ("id2", "B", 0.8), ("id3", ["A", "C"], [0.7, 0.3])]
```

You can also use the `evaluate` method to test the model against ground truth data. This returns an overall score for exact matches, partial matches, and false positives.

```
m.evaluate(ds, gt)
# {"exact": 0.5, "partial": 0.5, "false_positives": 0.0}
```



Prompt could include the codebook.

The codebook ought to be modifed to include a better description of each catetory.

Get fully functioning system working with just those steps before adding RAG. Get the success rate first.


Could also do RAG on the 2022 responses and provide context.
Do this by getting embeddings for each response and then using a vector database to query for similar responses. Add the the whole row to the context under "Similar responses". Indicate that examples use an older version of the codebook, so if uncertian follow the description in the codebook.

Instruct to label 2 if not clear what the correct responce is supposed to be.

for codebook 3.0, should send to llm with code names masked but descriptions included and examples, and ask to create new simple code names. the 1.0 names are not good or clear. using code names instead of numbers may have performance benefits.


After reading [this](https://doi.org/10.1177/20531680241231468), we might want to use text codes instead of numeric codes.

Followed [this course](https://learn.deeplearning.ai/courses/introducing-multimodal-llama-3-2) for prompting using proper chat tokens.



Could also add another method for classification of dataset that uses pure vector-based classification.


Can be used for other classification tasks.
Like:
Request for comment on policy
Get valence (+ or -) and category of concern


Review example articles that do AI classification and validate against human coders.

Example:

> Appendix B.2 provides examples of the resulting annotation. To validate this method, we compare the answers provided by GPT-engine to those provided by two independent research assistants for a random sample of 300 articles. Figure B2 shows that the agreement between Chat-GPT and a given human annotator is very similar to the agreement between two human annotators. We measure agreement by an accuracy score, i.e. the ratio of answers that are classified identically by GPT-engine and by the human annotator over the number of total answers. This lends confidence in the reliability of the method for this specific annotation task.23
https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4680430#page=47.86