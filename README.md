##### Original ([llama.cpp rocm port](https://github.com/ggerganov/llama.cpp/pull/1087), [llama.cpp commit](https://github.com/ggerganov/llama.cpp/commit/6bbc598a632560cb45dd2c51ad403bda8723b629)) by SlyEcho, YellowRoseCx, ardfork, funnbot, Engininja2, Kerfuffle, jammm, and jdecourval.
##### Further modified and ported to KoboldCpp by YellowRoseCx.
# koboldcpp-ROCM for AMD
### Quick Linux install:
To install, either use the file "[easy_KCPP-ROCm_install.sh](https://github.com/YellowRoseCx/koboldcpp-rocm/blob/main/easy_KCPP-ROCm_install.sh)" or navigate to the folder you want to download to in Terminal then run
```
git clone https://github.com/YellowRoseCx/koboldcpp-rocm.git -b main --depth 1 && \
cd koboldcpp-rocm && \
make LLAMA_HIPBLAS=1 -j4 && \
chmod +x koboldcpp.sh && ./koboldcpp.sh
```
When the KoboldCPP GUI appears, make sure to select "Use hipBLAS (ROCm)" and set GPU layers

KoboldCpp-ROCm is an easy-to-use AI text-generation software for GGML and GGUF models. It's an AI inference software from Concedo, maintained for AMD GPUs using ROCm by YellowRose, that builds off llama.cpp, and adds a versatile Kobold API endpoint, additional format support, Stable Diffusion image generation, backward compatibility, as well as a fancy UI with persistent stories, editing tools, save formats, memory, world info, author's note, characters, scenarios and everything KoboldAI and KoboldAI Lite have to offer.

![Preview](media/preview.png)
![Preview](media/preview2.png)
![Preview](media/preview3.png)
![Preview](media/preview4.png)

## Linux
- You will have to compile your binaries from source. A makefile is provided, simply run `make`.
- Arch Linux users can install koboldcpp via the AUR package provided by @AlpinDale. Please see [below](#arch-linux) for more details.
- For an ROCm only build, do ``make LLAMA_HIPBLAS=1 -j4`` (-j4 means it will use 4 cores of your CPU; you can adjust accordingly or leave it off altogether)
- Alternatively, if you want a full-featured build, you can also link CLBlast and or OpenBLAS by adding `LLAMA_CLBLAST=1 LLAMA_OPENBLAS=1` to the make command, for this you will need to obtain and link OpenCL and CLBlast libraries.
  - For Arch Linux: Install `cblas` `openblas` and `clblast`.
  - For Debian: Install `libclblast-dev` and `libopenblas-dev`.
- For a full featured build, do `make LLAMA_OPENBLAS=1 LLAMA_CLBLAST=1 LLAMA_HIPBLAS=1 -j4`
- After all binaries are built, you can use the GUI with ``python koboldcpp.py`` and select hipBLAS or run use ROCm through the python script with the command `python koboldcpp.py --usecublas --gpulayers [number] --contextsize 4096 --model [model.gguf]`
- There are several parameters than can be added to CLI launch, I recommend using ``--usecublas mmq`` or ``--usecublas mmq lowvram`` as it uses optimized Kernels instead of the generic rocBLAS code.
My typical start command looks like this: ``python koboldcpp.py --threads 6 --blasthreads 6 --usecublas mmq lowvram --gpulayers 18 --blasbatchsize 256 --contextsize 8192 --model /AI/llama-2-70b-chat.Q4_K_M.gguf``

## Windows Usage
- **[Download the latest .exe release here](https://github.com/YellowRoseCx/koboldcpp-rocm/releases/latest)** or clone the git repo.
- Windows binaries are provided in the form of **koboldcpp_rocm.exe**, which is a pyinstaller wrapper for a few **.dll** files and **koboldcpp.py**. You can also rebuild it yourself with the provided makefiles and scripts.
- Weights are not included, you can use the official llama.cpp `quantize.exe` to generate them from your official weight files (or download them from other places such as [TheBloke's Huggingface](https://huggingface.co/TheBloke).
- To run, simply execute **koboldcpp_rocm.exe**.
- Launching with no command line arguments displays a GUI containing a subset of configurable settings. Generally you dont have to change much besides the `Presets` and `GPU Layers`. Read the `--help` for more info about each settings.
- By default, you can connect to http://localhost:5001
- You can also run it using the command line. For info, please check `koboldcpp.exe --help` or `python koboldcpp.py --help`

- **AMD GPU Acceleration**: If you're on Windows with an AMD GPU you can get CUDA/ROCm HIPblas support out of the box using the `--usecublas` flag.
- **GPU Layer Offloading**: Want even more speedup? Combine one of the above GPU flags with `--gpulayers` to offload entire layers to the GPU! **Much faster, but uses more VRAM**. Experiment to determine number of layers to offload, and reduce by a few if you run out of memory.
- **Increasing Context Size**: Try `--contextsize 4096` to 2x your context size! without much perplexity gain. Note that you'll have to increase the max context in the KoboldAI Lite UI as well (click and edit the number text field).
- If you are having crashes or issues, you can try turning off BLAS with the `--noblas` flag. You can also try running in a non-avx2 compatibility mode with `--noavx2`. Lastly, you can try turning off mmap with `--nommap`.

For more information, be sure to run the program with the `--help` flag, or [check the wiki](https://github.com/LostRuins/koboldcpp/wiki).


## Compiling for AMD on Windows
(More in depth guide below)
  - Use the latest release of w64devkit (https://github.com/skeeto/w64devkit). Be sure to use the "vanilla one", not i686 or other different stuff. If you try they will conflit with the precompiled libs!
  - Make sure you are using the w64devkit integrated terminal, (powershell should work for the cmake hipblas part)
  - *This site may be useful, it has some patches for Windows ROCm to help it with compilation that I used, but I'm not sure if it's necessary with ROCm v5.7.* https://streamhpc.com/blog/2023-08-01/how-to-get-full-cmake-support-for-amd-hip-sdk-on-windows-including-patches/
  - (ROCm Required): https://rocm.docs.amd.com/en/latest/deploy/windows/quick_start.html

   Build command used:
  ``cd koboldcpp-rocm``
  ``mkdir build && cd build``

  ```cmake .. -G "Ninja" -DCMAKE_BUILD_TYPE=Release -DLLAMA_HIPBLAS=ON -DHIP_PLATFORM=amd -DCMAKE_C_COMPILER="C:/Program Files/AMD/ROCm/5.7/bin/clang.exe" -DCMAKE_CXX_COMPILER="C:/Program Files/AMD/ROCm/5.7/bin/clang++.exe" -DAMDGPU_TARGETS="gfx803;gfx900;gfx906;gfx908;gfx90a;gfx1010;gfx1030;gfx1031;gfx1032;gfx1100;gfx1101;gfx1102"```

  ``cmake --build . -j 6`` (-j 6 means use 6 CPU cores, if you have more or less, feel free to change it to speed things up)

  That puts koboldcpp_hipblas.dll inside of .\koboldcpp-rocm\build\bin
  copy koboldcpp_hipblas.dll to the main koboldcpp-rocm folder
  (You can run koboldcpp.py like this right away) like this:

  ``python koboldcpp.py --usecublas mmq --threads 1 --contextsize 4096 --gpulayers 45 C:\Users\YellowRose\llama-2-7b-chat.Q8_0.gguf``

  To make it into an exe, we use ``make_pyinstaller_exe_rocm_only.bat``
  which will attempt to build the exe for you and place it in /koboldcpp-rocm/dists/
  **kobold_rocm.exe is built!**
  ### More in depth AMD for Windows compilation guide:
  It seems like there's an issue with the commands you used because everything has escape slashes in it and the build process tries to use a different compiler than the one in the code before building the app

I'm not 100% sure if using w64devkit is still needed, using the Windows Terminal might work just as well.

Before starting,

1. Make sure when you type `python --version` it shows at least "*Python 3.10"*, I don't recommend using Python 3.11 or 3.12 for this; if its below Python 3.10, please upgrade Python. [Use the Windows Installer (64 bit) File on this page](https://www.python.org/downloads/release/python-31011/). After updating close out of Windows Terminal or w64devkit and reopen it, typing `python --version` should show "*Python 3.10.11*".
2. Make sure you have AMD ROCm 5.7 installed before doing this. Also, I recommend changing the "*AMDGPU\_TARGETS*" to only the one you're using as it will speed up compilation time significantly.
```
git clone https://github.com/YellowRoseCx/koboldcpp-rocm
cd koboldcpp-rocm
mkdir build && cd build
python -m pip install cmake ninja pyinstaller==6.4.0 psutil customtkinter

set CC=C:\Program Files\AMD\ROCm\5.7\bin\clang.exe
set CXX=C:\Program Files\AMD\ROCm\5.7\bin\clang++.exe
set CMAKE_PREFIX_PATH=C:\Program Files\AMD\ROCm\5.7
```
Then you can double check that CLang is set to the right version by using `clang --version` which should show "*AMD clang version 17.0.0*"

After that, you should be able to run the following build procedure:

    cmake .. -G "Ninja" -DCMAKE_BUILD_TYPE=Release -DLLAMA_HIPBLAS=ON -DHIP_PLATFORM=amd -DAMDGPU_TARGETS="gfx803;gfx900;gfx906;gfx908;gfx90a;gfx1010;gfx1030;gfx1031;gfx1032;gfx1100;gfx1101;gfx1102"
    cmake --build . --config Release -j2

Everything should be fine and dandy then. After that, you'll need to copy "*koboldcpp\_hipblas.dll*" from "\*koboldcpp-rocm\build\bin\koboldcpp\hipblas.dll*" to the main folder "/koboldcpp-rocm".

If you are using a AMD RX 6800 or 6900 variant or RX 7800 or 7900 variant, You should be able to run it directly with either `python koboldcpp.py` (for the GUI) or `python koboldcpp.py --usecublas mmq --threads 1 --contextsize 4096 --gpulayers 45 C:\Users\USERNAME\llama-2-7b-chat.Q8_0.gguf` for starting via command line.

If you have an AMD GPU that's a RX6600 or RX6700 variant, you'll either need to compile the ROCm tensile library files yourself (in lazy mode IIRC?) or use the community provided tensile libraries here: [https://github.com/YellowRoseCx/koboldcpp-rocm/releases/download/v1.43.2-ROCm/gfx103132rocblasfiles.zip](https://github.com/YellowRoseCx/koboldcpp-rocm/releases/download/v1.43.2-ROCm/gfx103132rocblasfiles.zip)

I've never done it *this* way, but if you're not compiling the exe, extract that zip file into *C:/Program Files/AMD/ROCm/5.7/bin/* it should merge with the "rocblas" folder that's already in there.

To build the exe, you *should* be able to use *"make\_pyinstaller\_exe\_rocm\_only.bat"* otherwise, make sure you're in the main /koboldcpp-rocm folder and you can do this:

    copy "C:\Program Files\AMD\ROCm\5.7\bin\hipblas.dll" .\ /Y
    copy "C:\Program Files\AMD\ROCm\5.7\bin\rocblas.dll" .\ /Y
    xcopy /E /I "C:\Program Files\AMD\ROCm\5.7\bin\rocblas" .\rocblas\ /Y

Then with that *gfx103132rocblasfiles.zip* file, extract the "rocblas" folder into /koboldcpp-rocm, the previous command will have copied the ROCm rocblas folder into /koboldcpp-rocm and you are merging the .zip files into that same folder. It can be done by hand, or by code like:

    curl -LO https://github.com/YellowRoseCx/koboldcpp-rocm/releases/download/v1.43.2-ROCm/gfx103132rocblasfiles.zip
    tar -xf gfx103132rocblasfiles.zip -C .\ --strip-components=1

Then you should be able to make the .exe file with this command:

    PyInstaller --noconfirm --onefile --clean --console --collect-all customtkinter --collect-all psutil --icon "./niko.ico" --add-data "./winclinfo.exe;." --add-data "./klite.embd;." --add-data "./kcpp_docs.embd;." --add-data="./kcpp_sdui.embd;." --add-data="./taesd.embd;." --add-data="./taesd_xl.embd;." --add-data "./rwkv_vocab.embd;." --add-data "./rwkv_world_vocab.embd;." --add-data "./koboldcpp_hipblas.dll;." --add-data "C:/Program Files/AMD/ROCm/5.7/bin/hipblas.dll;." --add-data "C:/Program Files/AMD/ROCm/5.7/bin/rocblas.dll;." --add-data "C:/Program Files/AMD/ROCm/5.7/bin/rocblas;." --add-data "C:/Windows/System32/msvcp140.dll;." --add-data "C:/Windows/System32/vcruntime140_1.dll;." "./koboldcpp.py" -n "koboldcpp_rocm.exe"



  -----
  If you'd like to do a full feature build with OPENBLAS and CLBLAST backends, you'll need [w64devkit](https://github.com/skeeto/w64devkit). Once downloaded, open w64devkit.exe and ``cd`` into the folder then run
  ``make LLAMA_OPENBLAS=1 LLAMA_CLBLAST=1 -j4`` then it will build the rest of the backend files.
  Once they're all built, you should be able to just run ``make_pyinst_rocm_hybrid_henk_yellow.bat`` and it'll bundle the files together into koboldcpp_rocm.exe in the \koboldcpp-rocm\dists folder
- If you wish to use your own version of the additional Windows libraries (OpenCL, CLBlast and OpenBLAS), you can do it with:
  - OpenCL - tested with https://github.com/KhronosGroup/OpenCL-SDK . If you wish to compile it, follow the repository instructions. You will need vcpkg.
  - CLBlast - tested with https://github.com/CNugteren/CLBlast . If you wish to compile it you will need to reference the OpenCL files. It will only generate the ".lib" file if you compile using MSVC.
  - OpenBLAS - tested with https://github.com/xianyi/OpenBLAS .
  - Move the respectives .lib files to the /lib folder of your project, overwriting the older files.
  - Also, replace the existing versions of the corresponding .dll files located in the project directory root (e.g. libopenblas.dll).
  - You can attempt a CuBLAS build with using the provided CMake file with visual studio. If you use the CMake file to build, copy the `koboldcpp_cublas.dll` generated into the same directory as the `koboldcpp.py` file. If you are bundling executables, you may need to include CUDA dynamic libraries (such as `cublasLt64_11.dll` and `cublas64_11.dll`) in order for the executable to work correctly on a different PC.
  - Make the KoboldCPP project using the instructions above.
  -

## Arch Linux Packages
There are some community made AUR packages (Maintained by @AlpinDale) available: [HIPBLAS](https://aur.archlinux.org/packages/koboldcpp-hipblas) and [CUBLAS](https://aur.archlinux.org/packages/koboldcpp-cuda).

The recommended installation method is through an AUR helper such as [paru](https://aur.archlinux.org/packages/paru) or [yay](https://aur.archlinux.org/packages/yay):

```sh
paru -S koboldcpp-cpuhipblas
```

Alternatively, you can manually install, though it's not recommended (since the build depends on [customtkinter](https://aur.archlinux.org/packages/customtkinter)):

```sh
git clone https://aur.archlinux.org/koboldcpp-hipblas.git && cd koboldcpp-hipblas

makepkg -si
```

You can then run koboldcpp anywhere from the terminal by running `koboldcpp` to spawn the GUI, or `koboldcpp --help` to view the list of commands for commandline execution (in case the GUI does not work).


## Android (Termux) Alternative method
- See https://github.com/ggerganov/llama.cpp/pull/1828/files

## Compiling on Android (Termux Installation)
- [Install and run Termux from F-Droid](https://f-droid.org/en/packages/com.termux/)
- Enter the command `termux-change-repo` and choose `Mirror by BFSU`
- Install dependencies with `pkg install wget git python` (plus any other missing packages)
- Install dependencies `apt install openssl` (if needed)
- Clone the repo `git clone https://github.com/LostRuins/koboldcpp.git`
- Navigate to the koboldcpp folder `cd koboldcpp`
- Build the project `make`
- Grab a small GGUF model, such as `wget https://huggingface.co/concedo/KobbleTinyV2-1.1B-GGUF/resolve/main/KobbleTiny-Q4_K.gguf`
- Start the python server `python koboldcpp.py --model KobbleTiny-Q4_K.gguf`
- Connect to `http://localhost:5001` on your mobile browser
- If you encounter any errors, make sure your packages are up-to-date with `pkg up`
- GPU acceleration for Termux may be possible but I have not explored it. If you find a good cross-device solution, do share or PR it.

## AMD
- Please check out https://github.com/YellowRoseCx/koboldcpp-rocm (you're already here 😊)

### Improving Performance
- **(Nvidia Only) GPU Acceleration**: If you're on Windows with an Nvidia GPU you can get CUDA support out of the box using the `--usecublas` flag, make sure you select the correct .exe with CUDA support.
- **Any GPU Acceleration**: As a slightly slower alternative, try CLBlast with `--useclblast` flags for a slightly slower but more GPU compatible speedup.
- **GPU Layer Offloading**: Want even more speedup? Combine one of the above GPU flags with `--gpulayers` to offload entire layers to the GPU! **Much faster, but uses more VRAM**. Experiment to determine number of layers to offload, and reduce by a few if you run out of memory.
- **Increasing Context Size**: Try `--contextsize 4096` to 2x your context size! without much perplexity gain. Note that you'll have to increase the max context in the Kobold Lite UI as well (click and edit the number text field).
- **Reducing Prompt Processing**: Try the `--smartcontext` flag to reduce prompt processing frequency.
- If you are having crashes or issues, you can try turning off BLAS with the `--noblas` flag. You can also try running in a non-avx2 compatibility mode with `--noavx2`. Lastly, you can try turning off mmap with `--nommap`.

For more information, be sure to run the program with the `--help` flag, or [check the wiki](https://github.com/LostRuins/koboldcpp/wiki).

## Run on Colab
- KoboldCpp now has an **official Colab GPU Notebook**! This is an easy way to get started without installing anything in a minute or two. [Try it here!](https://colab.research.google.com/github/LostRuins/koboldcpp/blob/concedo/colab.ipynb).
- Note that KoboldCpp is not responsible for your usage of this Colab Notebook, you should ensure that your own usage complies with Google Colab's terms of use.

## Docker
- The official docker can be found at https://hub.docker.com/r/koboldai/koboldcpp
- KoboldCpp also has a few unofficial third-party community created docker images. Feel free to try them out, but do not expect up-to-date support:
  - https://github.com/korewaChino/koboldCppDocker
  - https://github.com/noneabove1182/koboldcpp-docker
- If you're building your own docker, remember to set CUDA_DOCKER_ARCH or enable LLAMA_PORTABLE

## Questions and Help
- **First, please check out [The KoboldCpp FAQ and Knowledgebase](https://github.com/LostRuins/koboldcpp/wiki) which may already have answers to your questions! Also please search through past issues and discussions.**
- If you cannot find an answer, open an issue on this github, or find us on the [KoboldAI Discord](https://koboldai.org/discord).

## Considerations
- For Windows: No installation, single file executable, (It Just Works)
- Since v1.0.6, requires libopenblas, the prebuilt windows binaries are included in this repo. If not found, it will fall back to a mode without BLAS.
- Since v1.15, requires CLBlast if enabled, the prebuilt windows binaries are included in this repo. If not found, it will fall back to a mode without CLBlast.
- Since v1.33, you can set the context size to be above what the model supports officially. It does increases perplexity but should still work well below 4096 even on untuned models. (For GPT-NeoX, GPT-J, and LLAMA models) Customize this with `--ropeconfig`.
- Since v1.42, supports GGUF models for LLAMA and Falcon
- Since v1.55, lcuda paths on Linux are hardcoded and may require manual changes to the makefile if you do not use koboldcpp.sh for the compilation.
- Since v1.60, provides native image generation with StableDiffusion.cpp, you can load any SD1.5 or SDXL .safetensors model and it will provide an A1111 compatible API to use.
- **I plan to keep backwards compatibility with ALL past llama.cpp AND alpaca.cpp models**. But you are also encouraged to reconvert/update your models if possible for best results.

## ROCm vs OpenCL (old comparison)
Comparison with OpenCL using 6800xt (old measurement)
| Model | Offloading Method | Time Taken - Processing 593 tokens| Time Taken - Generating 200 tokens| Total Time | Perf. Diff.
|-----------------|----------------------------|--------------------|--------------------|------------|---|
| Robin 7b q6_K |CLBLAST 6-t, All Layers on GPU | 6.8s (11ms/T) | 12.0s (60ms/T)  | 18.7s (10.7T/s) | 1x
| Robin 7b q6_K |ROCM 1-t, All Layers on GPU   | 1.4s (2ms/T) | 5.5s (28ms/T)   | 6.9s (29.1T/s)| **2.71x**
| Robin 13b q5_K_M |CLBLAST 6-t, All Layers on GPU | 10.9s (18ms/T) | 16.7s (83ms/T)  | 27.6s (7.3T/s) | 1x
| Robin 13b q5_K_M |ROCM 1-t, All Layers on GPU   | 2.4s (4ms/T) | 7.8s (39ms/T)   | 10.2s (19.6T/s)| **2.63x**
| Robin 33b q4_K_S |CLBLAST 6-t, 46/63 Layers on GPU | 23.2s (39ms/T) | 48.6s (243ms/T)  | 71.9s (2.8T/s) | 1x
| Robin 33b q4_K_S |CLBLAST 6-t, 50/63 Layers on GPU | 25.5s (43ms/T) | 44.6s (223ms/T)  | 70.0s (2.9T/s) | 1x
| Robin 33b q4_K_S |ROCM 6-t, 46/63 Layers on GPU   | 14.6s (25ms/T) | 44.1s (221ms/T)   | 58.7s (3.4T/s)| **1.19x**

## License
- The original GGML library and llama.cpp by ggerganov are licensed under the MIT License
- However, KoboldAI Lite is licensed under the AGPL v3.0 License
- The other files are also under the AGPL v3.0 License unless otherwise stated

## Notes
- Generation delay scales linearly with original prompt length. If OpenBLAS is enabled then prompt ingestion becomes about 2-3x faster. This is automatic on windows, but will require linking on OSX and Linux. CLBlast speeds this up even further, and `--gpulayers` + `--useclblast` or `--usecublas` more so.
- I have heard of someone claiming a false AV positive report. The exe is a simple pyinstaller bundle that includes the necessary python scripts and dlls to run. If this still concerns you, you might wish to rebuild everything from source code using the makefile, and you can rebuild the exe yourself with pyinstaller by using `make_pyinstaller.bat`
- API documentation available at `/api` and https://lite.koboldai.net/koboldcpp_api
- Supported GGML models (Includes backward compatibility for older versions/legacy GGML models, though some newer features might be unavailable):
  - All up-to-date GGUF models are supported (Mistral/Mixtral/QWEN/Gemma and more)
  - LLAMA and LLAMA2 (LLaMA / Alpaca / GPT4All / Vicuna / Koala / Pygmalion 7B / Metharme 7B / WizardLM and many more)
  - GPT-2 / Cerebras
  - GPT-J
  - RWKV
  - GPT-NeoX / Pythia / StableLM / Dolly / RedPajama
  - MPT models
  - Falcon (GGUF only)
  - [Stable Diffusion and SDXL models](https://github.com/LostRuins/koboldcpp/wiki#can-i-generate-images-with-koboldcpp)

