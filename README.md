# Langchain-Chatchat with BigDL-LLM Acceleration on Intel GPUs

*Langchain-Chatchat* is a **RAG** (Retrieval Augmented Generation) application that implements knowledge and search engine based QA. This repo is a fork of [chatchat-space/Langchain-Chatchat](https://github.com/chatchat-space/Langchain-Chatchat), and includes [BigDL-LLM](https://github.com/intel-analytics/bigdl) optimizations to run it on **Intel GPU** (e.g., local PC with iGPU, discrete GPU such as Arc, Flex and Max). 
>You can change the UI language in the left-side menu. We currently support **English** and **简体中文** (see video demos below). 

<br/>

<table width="100%">
  <tr>
    <td align="center" width="50%"><code>English</code></td>
    <td align="center" width="50%"><code>简体中文</code></td>
  </tr>
  <tr>
    <td>
      <video src="https://github.com/intel-analytics/Langchain-Chatchat/assets/1995599/92bc5697-f633-4b26-b47f-1914228c659a" alt="english-video">english</video>
    </td>
    <td>
      <video src="https://github.com/intel-analytics/Langchain-Chatchat/assets/1995599/709bdc4d-dff9-45fa-bd55-90879ff8a5a8" alt="chinese-video">chinese</video>
    </td>
  </tr>
</table>

<br/>

The following sections introduce how to install and run Langchain-chatchat on **Intel Core Ultra platform (MTL)**, utilizing the iGPU to run both LLMs and embedding models. 

## Table of Contents
1. [RAG Architecture](langchain-chatchat-architecture)
2. [Installation](#installation)
3. [One-time Warmup](#one-time-warm-up)
4. [Start the Service](#start-the-service)
5. [How to Use](#usage)

## Langchain-Chatchat Architecture

See the RAG pipeline in the Langchain-Chatchat architecture below ([source](https://github.com/chatchat-space/Langchain-Chatchat/blob/master/img/langchain%2Bchatglm.png)).

<img src="https://github.com/chatchat-space/Langchain-Chatchat/blob/master/img/langchain%2Bchatglm.png" height="350px">
  
## Installation

### Download Langchain-Chatchat

Download the Langchain-Chatchat with BigDL-LLM integrations from [this link](https://github.com/intel-analytics/Langchain-Chatchat/archive/refs/heads/bigdl-llm.zip). Unzip the content into a directory, e.g.,`C:\Users\arda\Downloads\Langchain-Chatchat-bigdl-llm`. 

### Install Prerequisites

Visit the [Install BigDL-LLM on Windows with Intel GPU Guide](https://bigdl.readthedocs.io/en/latest/doc/LLM/Quickstart/install_windows_gpu.html), and follow [**Install Prerequisites**](https://bigdl.readthedocs.io/en/latest/doc/LLM/Quickstart/install_windows_gpu.html#install-prerequisites) to install Visual Studio, GPU driver, oneAPI, and Conda.  

### Install Python Dependencies

1.  Open **Anaconda Prompt (miniconda3)**, and run the following commands to create a new python environment:
    ```cmd
    conda create -n bigdl-langchain-chatchat python=3.11 libuv 
    conda activate bigdl-langchain-chatchat
    ```
    > **Note**: When creating the conda environment we used python 3.11, which is different from the default recommended python version 3.9 in [Install BigDL-LLM on Windows with Intel GPU](https://bigdl.readthedocs.io/en/latest/doc/LLM/Quickstart/install_windows_gpu.html)
    > 
2.  Install `bigdl-llm` 
    ```cmd
    pip install --pre --upgrade bigdl-llm[xpu] -f https://developer.intel.com/ipex-whl-stable-xpu
    pip install --pre --upgrade torchaudio==2.1.0a0  -f https://developer.intel.com/ipex-whl-stable-xpu
    ```
3. Switch to the root directory of Langchain-Chatchat you've downloaded (refer to the [download section](#download-langchain-chatchat)), and install the dependencies with the commands below. **Note: In the example commands we assume the root directory is `C:\Users\arda\Downloads\Langchain-Chatchat-bigdl-llm`. Remember to change it to your own path**).
    ```cmd
    cd C:\Users\arda\Downloads\Langchain-Chatchat-bigdl-llm
    pip install -r requirements_bigdl.txt 
    pip install -r requirements_api_bigdl.txt
    pip install -r requirements_webui.txt
    ```

### Configuration
-  In root directory of Langchain-Chatchat, run the following command to create a config:
    ```bash
    python copy_config_example.py
    ```
- Edit the file `configs\model_config.py`, change `MODEL_ROOT_PATH` to the absolute path where you put the downloaded models (LLMs, embedding models, ranking models, etc.)

### Download Models
Download the models and place them in the path `MODEL_ROOT_PATH` (refer to details in [Configuration](#configuration) section). 

Currently, we support only the LLM/embedding models specified in the table below. You can download these models using the link provided in the table. **Note: Ensure the model folder name matches the last segment of the model ID following "/", for example, for `THUDM/chatglm3-6b`, the model folder name should be `chatglm3-6b`.**


| Model |Category| download link | 
|:--|:--|:--|
|`THUDM/chatglm3-6b`|Chinese LLM| [HF](https://huggingface.co/THUDM/chatglm3-6b) or [ModelScope](https://www.modelscope.cn/models/ZhipuAI/chatglm3-6b/summary) |
|`meta-llama/Llama-2-7b-chat-hf`|English LLM| [HF](https://huggingface.co/meta-llama/Llama-2-7b-chat-hf) | 
|`BAAI/bge-large-zh-v1.5`|Chinese Embedding| [HF](https://huggingface.co/BAAI/bge-large-zh-v1.5) |
|`BAAI/bge-large-en-v1.5`| English Embedding|[HF](https://huggingface.co/BAAI/bge-large-en-v1.5) |

## One-time Warm-up
When you run this applcation on Intel GPU for the first time, it is highly recommended to do a one-time warmup (for GPU kernels compilation). 

In **Anaconda Prompt (miniconda3)**, under the root directory of Langchain-Chatchat, with conda environment activated, run the following commands:

```cmd
python warmup.py
```

> **Note**: The warmup may take several minutes. You just have to run it one-time on after installation. 

## Start the Service
 Open **Anaconda Prompt (miniconda3)** and run the following commands:
```cmd
conda activate bigdl-langchain-chatchat
call "C:\Program Files (x86)\Intel\oneAPI\setvars.bat"
set SYCL_CACHE_PERSISTENT=1
set BIGDL_LLM_XMX_DISABLED=1
set no_proxy=localhost,127.0.0.1
python startup.py -a
```
You can find the Web UI's URL printted on the terminal logs, e.g. http://localhost:8501/.

Open a browser and navigate to the URL to use the Web UI. 

## Usage

To start chatting with LLMs, simply type your messages in the textbox at the bottom of the UI. 

### How to use RAG

#### Step 1: Create Knowledge Base

- Select `Manage Knowledge Base` from the menu on the left, then choose `New Knowledge Base` from the dropdown menu on the right side.
  <p align="center"><img src="img/new-kb.png" alt="image1" width="70%" align="center"></p>
- Fill in the name of your new knowledge base (example: "test") and press the `Create` button. Adjust any other settings as needed. 
  <p align="center"><img src="img/create-kb.png" alt="image1" width="70%" align="center"></p>
- Upload knowledge files from your computer and allow some time for the upload to complete. Once finished, click on `Add files to Knowledge Base` button to build the vector store. Note: this process may take several minutes.
  <p align="center"><img src="img/build-kb.png" alt="image1" width="70%" align="center"></p>


#### Step 2: Chat with RAG

You can now click `Dialogue` on the left-side menu to return to the chat UI. Then in `Knowledge base settings` menu, choose the Knowledge Base you just created, e.g, "test". Now you can start chatting. 

<p align="center"><img src="img/rag-menu.png" alt="rag-menu" width="60%" align="center"></p>

<br/>

For more information about how to use Langchain-Chatchat, refer to Official Quickstart guide in [English](./README_en.md), [Chinese](./README_chs.md), or the [Wiki](https://github.com/chatchat-space/Langchain-Chatchat/wiki/).






