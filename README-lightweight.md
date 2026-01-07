# ComfyUI Wan Lightweight VACE Video Joiner

[Github](https://github.com/stuttlepress/ComfyUI-Wan-VACE-Video-Joiner) | [CivitAI](https://civitai.com/models/2024299)

This workflow uses Wan VACE to smooth out awkward motion transitions between video clips. If you have noisy frames at the start or end of your clips, this technique can also get rid of those.

## What it Does
This is a lightweight, (almost) no custom nodes ComfyUI workflow meant to quickly join two videos together with VACE and a minimum of fuss. Just load your videos and click Run. 

If you need to automatically join a large number of clips, mitigation of color/brightness artifacts, optimization options, try the full workflow instead.

## Dependencies
My [ComfyUI-Wan-VACE-Prep](https://github.com/stuttlepress/Comfyui-Wan-VACE-Prep) custom node is required for this workflow. It replaces a large amount of awkward spaghetti workflow math, making this lightweight workflow version possible.
In ComfyUI Manager, search for *Wan VACE Prep* or just load the workflow and then visit the **Missing** tab in Manager.
This is a very lightweight custom node with no dependencies, so it is highly unlikely to break your system, if that is something you worry about.

**I have not tested this workflow or the custom node under the Nodes 2.0 UI.**

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
- If you can't make the workflow work, update ComfyUI and try again. If you're not willing to update ComfyUI, I can't help you. We have to be working from the same starting point.
- Feel free to [open an issue on github](https://github.com/stuttlepress/ComfyUI-Wan-VACE-Video-Joiner/issues). This is the most direct way to engage me. If you want a head start, paste your complete console log from a failed run into your issue.

## Changelog
- **v1.0.3** Image Batch node fix.  
I don't quite understand the sequence of events, but since I updated ComfyUI to 0.7.0, the *Batch Images* node is back to its old static self. The dynamic behavior it had when I released this workflow is gone. I don't *think* I accidentally used a custom version of that node previously, but that is actually the most likely explanation. Anyway, this is hopefully the end of `Prompt outputs failed validation: BatchImagesNode: - Required input is missing: image1` messages.

- **v1.0.2** Core custom node change.
  - The VACE node I began with wasn't quite flexible enough, so I moved to [my own custom node](https://github.com/stuttlepress/ComfyUI-Wan-VACE-Prep) instead.  

- **v1.0.0** Initial release. 