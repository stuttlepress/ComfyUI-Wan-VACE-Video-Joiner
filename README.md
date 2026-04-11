# ComfyUI Wan VACE Video Joiner

[Github](https://github.com/stuttlepress/ComfyUI-Wan-VACE-Video-Joiner) | [CivitAI](https://civitai.com/models/2024299)

## What it Does

Point this workflow at a directory of clips and it will automatically stitch them together. It's designed to work well with a few clips or. At each transition, Wan VACE generates new frames guided by context on both sides, replacing the seam with motion that flows naturally between the clips. Noisy or artifacted frames at clip boundaries get replaced in the same pass. How many context frames and generated frames are used is configurable.

The workflow runs with either Wan 2.1 VACE or Wan 2.2 Fun VACE. Input clips can come from anywhere — Wan, LTX-2, phone footage, stock video, whatever you have. 

If you want the result to loop cleanly, there's a toggle for that.

## Usage

1. Put your input clips in their own directory, named so they sort in the order you want them joined.
2. Configure the workflow parameters. The notes in the workflow have full details on each one.
3. Set the index to 0.
4. Queue the workflow. **You need to queue it once per transition.** That's N-1 times for N clips, or N times if looping is enabled.

## Dependencies
I've used native nodes and tried to keep the custom node dependencies to a minimum. The following packages are required. All of them are installable through the Manager.
- [ComfyUI-Wan-VACE-Prep](https://github.com/stuttlepress/ComfyUI-Wan-VACE-Prep) v1.0.12 or higher
- [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes)
- [ComfyUI-VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite)
- [Basic data handling](https://github.com/StableLlama/ComfyUI-basic_data_handling)
- [ComfyUI-mxToolkit](https://github.com/Smirnov75/ComfyUI-mxToolkit)

**Note:** I have not tested this workflow under the ComfyUI Nodes 2.0 renderer.

## Models and Configuration
Configure this workflow the same way you have your existing Wan generation workflow set up. What runs well there will run well here.

Inference is isolated in the *Sampler* subgraph so it's straightforward to swap in your preferred setup. Replace the provided subgraph with one that works for you and plug it in. Have your way with it until it feels right.

Just make sure all subgraph inputs and outputs are correctly wired, and crucially, that the diffusion model you load is *Wan 2.2 Fun VACE* or *Wan 2.1 VACE*. GGUFs work fine, but non-VACE models do not. An example alternate sampler subgraph for VACE 2.1 is included.

Enable sageattention and torch compile if your system supports them.


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
- **Double images / ghosting at transitions with Wan 2.1 VACE** - In at least some cases, Wan 2.1 VACE outputs fewer frames than were in the control video input, even with correctly-sized 4n+1 inputs. This causes frame misalignment during cross-fade, producing double images at transitions. No fix yet. If you hit this, disabling cross-fade is the workaround for now.
- **Regarding Framerate** - The Wan models are trained at 16 fps, so if your input videos are at some higher rate, you may get sub-optimal results. At the very least, you'll need to increase the number of context and replace frames by whatever factor your framerate is greater than 16 fps in order to achieve the same effect with VACE. I suggest forcing your inputs down to 16 fps for processing with this workflow, then re-interpolating back up to your desired framerate.
- **IndexError: list index out of range** - Your input video may be too small for the parameters you have specified. The minimum size for a video will be `(context_frames + replace_frames) * 2 + 1`. Confirm that all of your input videos have at least this minimum number of frames.
- **Out of memory at the video join step** - The final video combine step is system RAM intensive. If you have a lot of input videos, you could run out of memory here. If this happens, use the VideoHelperSuite Meta Batch Manager to accomplish the combine in smaller chunks. A note in the workflow goes into more detail about exactly how to do this.
- If you can't make the workflow work, update ComfyUI and try again. If you're not willing to update ComfyUI, I can't help you. We have to be working from the same starting point.
- Feel free to [open an issue on github](https://github.com/stuttlepress/ComfyUI-Wan-VACE-Video-Joiner/issues). This is the most direct way to engage me. If you want a head start, paste your complete console log from a failed run into your issue.

---
## Examples
All examples are a series of Wan 2.2 first-last frame generations joined together with this workflow and Wan 2.2 Fun VACE.

https://github.com/user-attachments/assets/2e7f7cdb-0ce3-4379-a96d-7cf25e28a1ed  

https://github.com/user-attachments/assets/5057eafc-cc6c-4801-bf9c-53fcf3f0eb61

https://github.com/user-attachments/assets/13d7cacd-1404-49a1-8652-6bfdf49efdf3

---

## Changelog
- **v2.5**
  - **Seamless Loops** - Enable the `Make Loop` toggle and the workflow will generate a smooth transition between your final input video and the first one, allowing the video to be played on a loop.
  - **Much lower RAM usage during final assembly** - Enabled by default, VideoHelperSuite's *Meta Batch Manager* drastically reduces the amount of system RAM consumed while concatenating frames. If you were running out of RAM on the final step because you were joining hundreds or thousands of frames, that shouldn't be a  problem any more. Additional details in the workflow notes.
- **v2.4** - Minor tweaks. Adjust sage attention, torch compile defaults.
- **v2.3** This release prioritizes workflow reliability and maintainability. Core functionality remains unchanged. These changes reduce surface area for failures and improve debuggability. Stability and deterministic operation take priority over convenience features.
  - **Looping workflow discontinued** - While still functional, the loop-based approach obscured workflow status and complicated targeted reruns for specific transitions. The batch workflow provides better visibility and control.
  - **Reverted to lossless fv1 intermediate files** - The 16-bit PNG experiment provided no practical benefit and made addressing individual joins more cumbersome. Returning to the proven method.
  - **New custom nodes for cleaner workflows** - *WAN VACE Prep Batch* and *VACE Batch Context* encapsulate operations that are awkward to express in visual nodes but straightforward in Python. *Load Videos From Folder (simple)* replaces the KJNodes equivalent to eliminate problematic VideoHelperSuite dependencies that fail in some environments.
  - **Enhanced console logging** - Additional diagnostic output when `Debug=True` to aid troubleshooting.
  - **Fewer custom node dependencies**

- **[Lightweight workflow added.](README-lightweight.md)**
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
