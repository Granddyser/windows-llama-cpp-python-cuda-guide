# windows-llama-cpp-python-cuda-guide
A comprehensive, step-by-step guide for successfully installing and running llama-cpp-python with CUDA GPU acceleration on Windows. This repository provides a definitive solution to the common installation challenges, including exact version requirements, environment setup, and troubleshooting tips. 

# Guide: llama-cpp-python with CUDA on Windows (Definitive & Corrected Method)

Since I couldn't find a comprehensive guide or a reliable solution to get `llama-cpp-python` running smoothly with CUDA on Windows, here's my consolidated approach. This method has successfully helped two of my friends overcome various installation issues by focusing on a clean, local environment and specific package versions.

## 🌟 Highlights of This Guide

*   **Local, Deletable Conda Environment:** Environment created directly in the project folder for easy cleanup (using `conda create --prefix ./env python=3.11`).
*   **Specific CUDA Toolkit Installation via Conda:** Uses `conda install nvidia/label/cuda-12.1.0::cuda-toolkit`.
*   **Precise PyTorch Installation for CUDA 12.1:** Uses `pip3 install torch --index-url https://download.pytorch.org/whl/cu121`.
*   **Precise Visual Studio 2019 Configuration:** Exact specification of required components with a direct download link.
*   **Specific Download Links:** For Visual Studio 2019, (system) CUDA Toolkit 12.1.0, and CMake 3.31.7.
*   **Detailed Environment Variable Setup:** For the system-level CUDA, which aids the C++ compiler.
*   **Troubleshooting:** Tips for cleaning up after failed attempts.
*   **Correct `llama-cpp-python` Installation:** Including necessary build arguments for CUDA.

## 0. Preparation: System Cleanup (Optional, but highly Recommended for Issues)

If you've had previous, unsuccessful installation attempts of `llama-cpp-python` or similar packages, it's advisable to remove potential remnants:

1.  **Uninstall Visual Studio 2022:** If VS 2022 is installed and causing problems, uninstall it via "Apps & Features" in Windows Settings. This guide focuses on VS 2019.
2.  **Delete Temporary Files and Caches:**
    *   Close all terminals and development environments.
    *   Open Windows Explorer and type `%TEMP%` into the address bar. Delete the contents of this folder (some files might be locked, which is okay).
    *   Type `%APPDATA%`. Look for folders related to `pip` or `cmake` (e.g., `pip/cache`) and delete their contents or the folders themselves if you're sure. Be cautious here.
    *   If necessary, delete old, faulty Conda environments (especially if they were not created with `--prefix` inside a project folder).

## 1. Prerequisites Check and Setup (System-Level)

Even though we install a CUDA toolkit into Conda, having a system-level CUDA Toolkit (especially for the C++ compiler and `nvcc` to be found easily by CMake/pip during build) and correct Visual Studio setup is crucial.

### 1.1. NVIDIA CUDA Version Check (System)
Check your NVIDIA drivers and any system-installed CUDA Toolkits.

*   **Driver & Supported CUDA Version (`nvidia-smi`):**
    Open PowerShell and enter:
    ```powershell
    nvidia-smi
    ```
    Note the "CUDA Version" in the top right. This is the *maximum* version supported by your current driver.
*   **Installed System CUDA Toolkits (`nvcc --version`):**
    If you already have a system-wide CUDA Toolkit installed:
    ```powershell
    nvcc --version
    ```
    This shows the version of the toolkit currently found in the system path. This guide assumes you'll have CUDA 12.1 system-wide for the compiler.

### 1.2. Visual Studio 2019 Installation and Configuration
`llama-cpp-python` requires a C++ compiler. Visual Studio 2019 is recommended.

1.  **Download:** Download **Visual Studio 2019 Community Edition** directly from archive.org:
    *   **[Download Visual Studio 2019 Community installer from archive.org](https://archive.org/details/vs_community__e8aae2bc1239469a8cb34a7eeb742747)**
   

2.  **Installation – Workloads & Components:**
    *   Select the workload **"Desktop development with C++"**.
    *   Go to **"Individual components"** and ensure these are selected:
        *   `MSVC v142 - VS 2019 C++ x64/x86 build tools (Latest)`
        *   `Windows 10 SDK (e.g., 10.0.19041.0)`
        *   **`Windows 11 SDK (e.g., 10.0.22000.0 or newer)` – Crucial!**
        *   `C++ CMake tools for Windows`
        **Example of selected components:**
    
    ![VS2019 Components Selection](images/image.png)

    *This screenshot shows the required components you need to select during installation.*
   


### 1.3. CMake Installation (System-Level)
*   **Download CMake Version 3.31.7:**
    *   **[Download cmake-3.31.7-windows-x86_64.msi from GitHub](https://github.com/Kitware/CMake/releases/download/v3.31.7/cmake-3.31.7-windows-x86_64.msi)**
*   During installation, enable **"Add CMake to the system PATH for all users"**.

### 1.4. System-Level CUDA Toolkit Installation (Version 12.1.0)
This system-level installation helps the C++ build tools find `nvcc`.

1.  **Download CUDA Toolkit 12.1.0:**
    *   **[Download CUDA Toolkit 12.1.0 from NVIDIA](https://developer.nvidia.com/cuda-12-1-0-download-archive)** (select Windows, x86_64, version 10/11, `exe (local)`).
2.  **Installation:** Choose "Custom (Advanced)", select "CUDA" (including Visual Studio Integration). Note the path: `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1`.

### 1.5. Setting and Verifying System CUDA Environment Variables
This is critical for `nvcc` to be found by the build process.

1.  **Open Environment Variables:** Search "Edit the system environment variables".
2.  **Set/Check `CUDA_PATH` and `CUDA_PATH_v_1_x_xx`(System Variable):**
    *   Name: `CUDA_PATH`
    *   Value: `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1`
3.  **Edit `Path` Variable (System Variables):**
    *   Ensure these are at the top:
        *   `%CUDA_PATH%\bin`
        *   `%CUDA_PATH%\libnvvp`
4.  **Apply Changes and Restart PowerShell:** Close ALL PowerShells and open a new one.
5.  **Verification in new PowerShell:**
    ```powershell
    echo $env:CUDA_PATH
    nvcc --version 
    # Should show 12.1
    ```

## 2. Create Project and Local Conda Environment in Powershellx64

1.  **Create Project Folder:**
    ```powershell
    mkdir D:\AI\LlamaCPPProject
    cd D:\AI\LlamaCPPProject
    ```

2.  **Create Local Conda Environment (named `env` inside project):**
    This is key for easy deletion if things go wrong.
    ```powershell
    # Important: Use the x64 version of PowerShell
    conda create --prefix ./env python=3.11
    ```

3.  **Activate Conda Environment:**
    ```powershell
    conda activate ./env 
    # Or from the project root: conda activate .\env
    ```
    Your PowerShell prompt should change to show `(./env)`.

## 3. Install CUDA Toolkit and PyTorch into Conda Environment

Now, inside the activated `env` environment:

1.  **Install CUDA Toolkit 12.1.0 via Conda:**
    This provides the CUDA runtime libraries specifically for this environment.
    ```powershell
    # Ensure (./env) is active
    conda install nvidia/label/cuda-12.1.0::cuda-toolkit -c nvidia/label/cuda-12.1.0
    ```
    *Note: The channel specification `-c nvidia/label/cuda-12.1.0` might be redundant if the package name already includes it, but it ensures the correct source.*

2.  **Install PyTorch for CUDA 12.1 (NO AUDIO, NO VISION):**
    This specific command installs PyTorch compiled for CUDA 12.1. `pip3` is often an alias for `pip` within Conda environments. Use `pip` if `pip3` is not found.
    ```powershell
    # Ensure (./env) is active
    pip3 install torch --index-url https://download.pytorch.org/whl/cu121
    # If pip3 gives an error, try:
    # pip install torch --index-url https://download.pytorch.org/whl/cu121
    ```

## 4. Install `llama-cpp-python` with CUDA

Still inside the activated `env` environment:

1.  **Set Build Arguments in PowerShell:**
    These instruct `pip` to compile `llama-cpp-python` with CUDA support (CUBLAS) and to use the system's `nvcc.exe` (found via `CUDA_PATH`).
    ```powershell
    # Ensure (./env) is active!
    $env:CMAKE_ARGS="-DLLAMA_CUBLAS=on"
    # This line is crucial for explicitly pointing to the CUDA compiler:
    $env:CUDA_CXX="C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\bin\nvcc.exe"
    ```

2.  **Installation via pip:**
    Use `pip` (or `pip3` if that's your preference and it works in your Conda env).
    ```powershell
    # Ensure (./env) is active
    pip install llama-cpp-python[server] --upgrade --force-reinstall --no-cache-dir
    ```
    This process might take some time. Look for "Successfully built llama-cpp-python".

## 5. Verify Installation

1.  **Start Python in your activated Conda environment (`env`):**
    ```powershell
    # Ensure (./env) is active
    python
    ```

2.  **Test PyTorch CUDA availability and `llama-cpp-python` import:**
    ```python
    import torch
    import os

    print(f"PyTorch version: {torch.__version__}")
    print(f"Is CUDA available for PyTorch? {torch.cuda.is_available()}")
    if torch.cuda.is_available():
        print(f"PyTorch CUDA version: {torch.version.cuda}")
        print(f"Number of GPUs PyTorch sees: {torch.cuda.device_count()}")
        if torch.cuda.device_count() > 0:
            print(f"Current GPU Model (PyTorch): {torch.cuda.get_device_name(0)}")
    
    print("\nAttempting to import Llama...")
    try:
        from llama_cpp import Llama
        print("Llama imported successfully!")
        # For a more thorough test, you'd load a model with n_gpu_layers > 0
        # llm = Llama(model_path="path/to/your.gguf", n_gpu_layers=30) 
        # print("Llama object initialized (this would test actual GPU offload).")
    except Exception as e:
        print(f"Error importing or initializing Llama: {e}")

    print("\nChecking CMAKE_ARGS from Python environment:")
    print(f"CMAKE_ARGS: {os.environ.get('CMAKE_ARGS')}") 
    
    quit()
    ```

## 6. Prepare Model and Run Application

1.  **Download Models:** Get GGUF models (e.g., from TheBloke on Hugging Face) and place them in a subfolder like `D:\AI\LlamaCPPProject\models`.

2.  **Start Application (Example with Python server):**
    ```powershell
    # Ensure (./env) is active!
    # Ensure $env:CMAKE_ARGS="-DLLAMA_CUBLAS=on" is set in this PowerShell session
    
    python -m llama_cpp.server --model D:\AI\LlamaCPPProject\models\YOUR_MODEL.gguf --n_gpu_layers -1 
    ```
    Monitor for GPU offload messages and check Task Manager for GPU activity.

---

## 💡 Bonus Strategy: Universal AI Project Compatibility ⚡⚡⚡

### Better CUDA Version Strategy for AI/ML Projects

While this guide uses **CUDA 12.1** (because it worked reliably for this specific setup), here's a more universal approach for Windows AI/ML development:

> **Recommendation:** Install **CUDA 11.8, 12.6, and 12.8** on your system. These three versions cover compatibility with almost every AI project you'll encounter.

### **Understanding PyTorch Installation Options**

**Two main approaches exist for PyTorch installation:**
- **Official PyTorch Builds:** Current official support for CUDA 11.8, 12.6, and 12.8 only
- **Pre-built Wheels:** Community/third-party wheels available for more CUDA versions (like 12.1, 12.4)

**Use Cases by CUDA Version:**
- **CUDA 11.8:** Legacy projects, older Stable Diffusion models, most GitHub repositories from 2022-2023
- **CUDA 12.6:** Current mainstream AI projects, latest PyTorch features, balanced compatibility  
- **CUDA 12.8:** Cutting-edge frameworks, latest GPU architectures (RTX 50xx series), experimental features

**Why These Specific Versions?**
- **CUDA 11.8:** Compatible with most older AI frameworks and models
- **CUDA 12.6:** Current stable PyTorch official support and modern projects  
- **CUDA 12.8:** Latest stable version for cutting-edge frameworks and newest GPUs


## Critical Setup Requirements:
1. **System Environment Variables:** 
   - Configure `CUDA_PATH`, `CUDA_HOME` pointing to your primary CUDA version
   - Add all CUDA `bin` directories to the system **"Path"** variable (found in System Environment Variables)

2. **Install PyTorch with exact version matching:**

   **For pip users (Official PyTorch builds - Recommended):**
   ```powershell
   # CUDA 11.8
   pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
   
   # CUDA 12.6  
   pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126
   
   # CUDA 12.8
   pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128
   ```

   **For conda users:**
   ```powershell
   # CUDA 11.8
   conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia

   # CUDA 12.1 (for this guide - uses pre-built wheels)
   conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia

   # CUDA 12.6
   conda install pytorch torchvision torchaudio pytorch-cuda=12.6 -c pytorch -c nvidia
   ```
   Result: With this setup, you can clone and run almost any AI project from GitHub without build issues or CUDA compatibility problems.

    ⚠️ I have CUDA 11.8, 12.4, and 12.8 installed on my system – you can install as many CUDA versions side by side as you like!
    ⚠️ [See critical setup requirements above](#critical-setup-requirements) on configuring your system environment variables (CUDA_PATH, CUDA_HOME, Path)!
    Always make sure these point to the CUDA version you want to use.

   This guide was created based on the information provided and the specified corrections.
   Last Updated: June 2025

   
