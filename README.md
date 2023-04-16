# koboldcpp (wordstopper fork)

A self contained distributable from Concedo that exposes llama.cpp function bindings, allowing it to be used via a simulated Kobold API endpoint. 

What does it mean? You get llama.cpp with a fancy UI, persistent stories, editing tools, save formats, memory, world info, author's note, characters, scenarios and everything Kobold and Kobold Lite have to offer. In a tiny package around 10 MB in size, excluding model weights.

![Preview](preview.png)

# Highlights
- Now has experimental CLBlast support.
- Now has a token stopper to speed up generation by reducing wasted tokens

## Usage
- Download or clone the repo https://github.com/YellowRoseCx/koboldcpp-wordstopper
- Windows users: Download the .rar file on the right side of the page under Releases then launch with "python koboldcpp.py ggml_model_name.bin"
- Linux and OSX instructions are next paragraph.
- To run, open command prompt or terminal in the koboldcpp-wordstopper directory then launch with "python koboldcpp.py models/ggml_model_name.bin" and then connect with Kobold or Kobold Lite. Please check `python koboldcpp.py --help` for more info
- By default, you can connect to http://localhost:5001 
- If you are having crashes or issues with OpenBLAS, please try the `--noblas` flag.
- Add the names or tokens you wish to use as stoppers to the wordstoppers.txt file. If it's a chat name, make sure to add the colon afterwards.


## OSX and Linux
- To link with your own install of OpenBLAS manually with `make LLAMA_OPENBLAS=1`
- Alternatively, if you want you can also link your own install of CLBlast manually with `make LLAMA_CLBLAST=1`, for this you will need to obtain and link OpenCL and CLBlast libraries.
  - For Arch Linux: Install `cblas` and `openblas`. In the makefile, find the `ifdef LLAMA_OPENBLAS` conditional and add `-lcblas` to `LDFLAGS`.
  - For Debian: Install `libclblast-dev` and `libopenblas-dev`. If you get a clbas_sgemm error, add -lcblas like in the Arch instructions.
- After all binaries are built, you can run the python script with the command `koboldcpp.py [ggml_model.bin] [port]`

## Considerations
- ZERO or MINIMAL changes as possible to parent repo files - do not move their function declarations elsewhere! We want to be able to update the repo and pull any changes automatically.
- No dynamic memory allocation! Setup structs with FIXED (known) shapes and sizes for ALL output fields. Python will ALWAYS provide the memory, we just write to it.
- Since v1.0.6, requires libopenblas, the prebuilt windows binaries are included in this repo. If not found, it will fall back to a mode without BLAS. 
- Since v1.15, requires CLBlast if enabled, the prebuilt windows binaries are included in this repo. If not found, it will fall back to a mode without CLBlast. 
- **I plan to keep backwards compatibility with ALL past llama.cpp AND alpaca.cpp models**. But you are also encouraged to reconvert/update your models if possible for best results.

## License
- The original GGML library and llama.cpp by ggerganov are licensed under the MIT License
- However, Kobold Lite is licensed under the AGPL v3.0 License
- The other files are also under the AGPL v3.0 License unless otherwise stated

## Notes
- Generation delay scales linearly with original prompt length. If OpenBLAS is enabled then prompt ingestion becomes about 2-3x faster. This is automatic on windows, but will require linking on OSX and Linux.
