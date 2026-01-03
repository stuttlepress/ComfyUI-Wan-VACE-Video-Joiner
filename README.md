# ComfyUI Wan VACE Video Joiner

[Github](https://github.com/stuttlepress/ComfyUI-Wan-VACE-Video-Joiner) | [CivitAI](https://civitai.com/models/2024299)

This workflow uses Wan VACE (Wan 2.2 Fun VACE or Wan 2.1 VACE, your choice!) to smooth out awkward motion transitions between video clips. If you have noisy frames at the start or end of your clips, this technique can also get rid of those.

I've used this workflow to join first-last frame videos for some time and I thought others might find it useful.

## What it Does
The workflow iterates over any number of video clips in a directory, generating smooth transitions between them by replacing a configurable number of frames at the transition. The frames found just before and just after the transition are used as context for generating the replacement frames. The number of context frames is also configurable. Optionally, the workflow can also join the smoothed clips together. Or you can accomplish this in your favorite video editor.

## Changelog
- **v2.2** *Complexity Reduction Release*
  - Removed fancy model loader which was causing headaches for safetensors users without any gguf models installed, and vice-versa.
  - Removed the MOE KSampler and TripleKSampler subgraphs. You can still use these samplers, but it's up to you to bring them and set them up.
  - Custom node dependencies reduced.
  - Un-subgraphed some functions. Sadly, this powerful and useful feature is still too unstable to distribute to users on varying versions of ComfyUI.

- **v2.1**
  - Add Prune Outputs to Video Combine nodes, preventing extra frames from being added to the output 

- **v2.0**
  - Workflow redesign. Core functionality is the same, but hopefully usability is improved
  - (Experimental) New looping workflow variant that doesn't require manual queueing and index manipulation. I am not entirely comfortable with this version and consider it experimental. The ComfyUI-Easy-Use For Loop implementation is janky and requires some extra, otherwise useless code to make it work. But it lets you run with one click! Use with caution. All VACE join features are identical between the workflows. Looping is the only difference.
  - (Experimental) Added cross fade at VACE boundaries to mitigate brightness/color shift
  - (Experimental) Added color match for VACE frames to mitigate brightness/color shift
  - Save intermediate work as 16 bit png instead of ffv1 to mitigate brightness/color shift
  - Integrated video join into the main workflow. It will run automatically after the last iteration. No more need to run the join part separately.
  - More documentation
  - Inputs and outputs are logged to the console for better progress tracking

- **v1.3**
  - removed lingering custom primitive node

- **v1.2**
  - Sort the input directory list.  

- **v1.1**
  - Preserve input framerate in workflow VACE outputs. Previously, all output was forced to 16fps. Note, you must manually set the framerate in the Join & Save output.  
  - Changed default model/sampler to Wan 2.2 Fun VACE fp8/KSampler. GGUF, MoE, 2.1 are still available in the bypassed subgraphs.

## Usage
***This is not a ready to run workflow. You need to configure it to fit your system.***
What runs well on my system will not necessarily run well on yours. Configure this workflow to use a VACE model of the same type that you use in your standard Wan workflow. Detailed configuration and usage instructions can be found in the workflow. Please read carefully.

## Dependencies
I've used native nodes and tried to keep the custom node dependencies to a minimum. The following packages are required. All of them are installable through the Manager.

- [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes)
- [ComfyUI-VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite)
- [Basic data handling](https://github.com/StableLlama/ComfyUI-basic_data_handling)
- [ComfyUI-mxToolkit](https://github.com/Smirnov75/ComfyUI-mxToolkit)
- [ComfyUI-WanVideoWrapper](https://github.com/kijai/ComfyUI-WanVideoWrapper) - new for version 2. Supplies a handy node that simplifies VACE control video creation.
- [ComfyUI-Easy-Use](https://github.com/yolain/ComfyUI-Easy-Use) - new for version 2. Only required for the loop version of the workflow. Needed for the For Loop nodes.
- [ComfyUI_essentials](https://github.com/cubiq/ComfyUI_essentials) - new for version 2. Used for logging to the console.

**I have not tested this workflow under the Nodes 2.0 UI.**

Model loading and inference is isolated in subgraphs, so It should be easy to modify this workflow for your preferred setup. Just replace the provided sampler subgraph with one that implements your stuff, then plug it into the workflow. A few example alternate sampler subgraphs, including one for VACE 2.1, are included.

I am happy to answer questions about the workflow. I am less happy to instruct you on the basics of ComfyUI usage.

## Configuration and Models
You'll need some combination of these models to run the workflow. As already mentioned, this workflow will not run properly on your system until you configure it properly. You probably already have a Wan video generation workflow that runs well on your system. You need to configure this workflow similarly to your generation workflow. The *Sampler* subgraph contains KSampler nodes and model loading nodes. Have your way with these until it feels right to you. Enable the sageattention and torch compile nodes if you know your system supports them. Just make sure all the subgraph inputs and outputs are correctly getting and setting data, and crucially, that the diffusion model you load is one of *Wan2.2 Fun VACE* or *Wan2.1 VACE*. GGUFs work fine, but non-VACE models do not.

- Wan 2.2 Fun VACE
  - [bf16 and fp8](https://huggingface.co/Comfy-Org/Wan_2.2_ComfyUI_Repackaged/tree/main/split_files/diffusion_models)
  - [GGUF](https://huggingface.co/QuantStack/Wan2.2-VACE-Fun-A14B-GGUF/tree/main)
- Wan 2.1 VACE
  - [fp16](https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/diffusion_models/wan2.1_vace_14B_fp16.safetensors?download=true)
  - [GGUF](https://huggingface.co/QuantStack/Wan2.1_14B_VACE-GGUF/tree/main)
- Kijai’s extracted Fun Vace 2.2 modules, for loading along with standard T2V models. [Native use examples here.](https://huggingface.co/Kijai/WanVideo_comfy/discussions/81)
  - [bf16](https://huggingface.co/Kijai/WanVideo_comfy/tree/main/Fun/VACE)
  - [GGUF](https://huggingface.co/Kijai/WanVideo_comfy_GGUF/tree/main/VACE)

## Troubleshooting
- **The size of tensor a must match the size of tensor b at non-singleton dimension 1** - Check that both dimensions of your input videos are divisible by 16 and change this if they're not. Fun fact: 1080 is not divisible by 16!
- **Brightness/color shift** - VACE can sometimes affect the brightness or saturation of the clips it generates. I don't know how to avoid this tendency, I think it's baked into the model, unfortunately. Disabling lightx2v speed loras can help, as can making sure you use the exact same lora(s) and strength in this workflow that you used when generating your clips. Some people have reported success using a color match node before output of the clips in this workflow. I think specific solutions vary by case, though. The most consistent mitigation I have found is to interpolate framerate up to 30 or 60 fps after using this workflow. The interpolation decreases how perceptible the color shift is. The shift is still there, but it's spread out over 60 frames instead over 16, so it doesn't look like a sudden change to our eyes any more.
- **Regarding Framerate** - The Wan models are trained at 16 fps, so if your input videos are at some higher rate, you may get sub-optimal results. At the very least, you'll need to increase the number of context and replace frames by whatever factor your framerate is greater than 16 fps in order to achieve the same effect with VACE. I suggest forcing your inputs down to 16 fps for processing with this workflow, then re-interpolating back up to your desired framerate.
- **IndexError: list index out of range** - Your input video may be too small for the parameters you have specified. The minimum size for a video will be `(context_frames + replace_frames) * 2 + 1`. Confirm that all of your input videos have at least this minimum number of frames.
- **Out of memory at the video join step** - The final video combine step is system RAM intensive. If you have a lot of input videos, you could run out of memory here. If this happens, use the VideoHelperSuite Meta Batch Manager to accomplish the combine in smaller chunks. A note in the workflow goes into more detail about exactly how to do this.
- If you can't make the workflow work, update ComfyUI and try again. If you're not willing to update ComfyUI, I can't help you. We have to be working from the same starting point.
- Feel free to [open an issue on github](https://github.com/stuttlepress/ComfyUI-Wan-VACE-Video-Joiner/issues). This is the most direct way to engage me. If you want a head start, paste your complete console log from a failed run into your issue.
