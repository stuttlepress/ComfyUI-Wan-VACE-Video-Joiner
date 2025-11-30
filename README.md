# ComfyUI Wan VACE Video Joiner

[Github](https://github.com/stuttlepress/ComfyUI-Wan-VACE-Video-Joiner) | [CivitAI](https://civitai.com/models/2024299) | [Reddit Discussion](https://www.reddit.com/r/comfyui/comments/1o0l5l7/wan_vace_clip_joiner_native_workflow)

This workflow uses Wan VACE (Wan 2.2 Fun VACE or Wan 2.1 VACE, your choice!) to smooth out awkward motion transitions between video clips. If you have noisy frames at the start or end of your clips, this technique can also get rid of those.

I've used this workflow to join first-last frame videos for some time and I thought others might find it useful.

## What it Does
The workflow iterates over any number of video clips in a directory, generating smooth transitions between them by replacing a configurable number of frames at the transition. The frames found just before and just after the transition are used as context for generating the replacement frames. The number of context frames is also configurable. Optionally, the workflow can also join the smoothed clips together. Or you can accomplish this in your favorite video editor.

## Usage
***This is not a ready to run workflow. You need to modify it to fit your system.***
What runs well on my system will not necessarily run well on yours. Configure this workflow to use the same model type and conditioning that you use in your standard Wan workflow. More detailed configuration and usage instructions can be found in the workflow. Please read this carefully.

## Dependencies
I've used native nodes and tried to keep the custom node dependencies to a minimum. The following packages are required. All of them are installable through the Manager.

- [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes)
- [ComfyUI-VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite)
- [ComfyUI-mxToolkit](https://github.com/Smirnov75/ComfyUI-mxToolkit)
- [Basic data handling](https://github.com/StableLlama/ComfyUI-basic_data_handling)
- [ComfyUI-GGUF](https://github.com/city96/ComfyUI-GGUF) - only needed if you'll be loading GGUF models. If not, you can delete the sampler subgraph that uses GGUF to remove the requirement.
- [KSampler for Wan 2.2. MoE for ComfyUI](https://github.com/stduhpf/ComfyUI-WanMoeKSampler) - only needed if you plan to use the MoE KSampler. If not, you can delete the MoE sampler subgraph to remove the requirement.

The workflow uses subgraphs, so your ComfyUI needs to be relatively up-to-date.

I have not tested this workflow under the new Nodes 2.0 UI.

Model loading and inference is isolated in a subgraph, so It should be easy to modify this workflow for your preferred setup. Just replace the provided sampler subgraph with one that implements your stuff, then plug it into the workflow.

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
