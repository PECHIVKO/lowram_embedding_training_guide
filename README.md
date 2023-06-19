# 6GB VRAM Textual Inversion Embedding Training Guide

I am writing this guiide because I couldn't find any examples of running embedding trainings on low VRAM setups. I am limited to using only the NVIDIA GeForce RTX 2060 in my laptop, so for this purpose, I used the [kohya_ss](https://github.com/bmaltais/kohya_ss) and [automatic1111 webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) repositories. Kohya_ss dreamboth ti allows me to get consistent results in different models after 2-3 hours of training. I referred to [this guide](https://www.reddit.com/r/StableDiffusion/comments/zxkukk/detailed_guide_on_training_embeddings_on_a/?utm_source=share&utm_medium=android_app&utm_name=androidcss&utm_term=1&utm_content=share_button) and [this guide](https://docs.google.com/document/u/0/d/1JvlM0phnok4pghVBAMsMq_-Z18_ip_GXvHYE0mITdFE/mobilebasic) while setting up my settings, and you can find a lot of useful information to understand and improve the process.

1. Install repositories: You can follow the links to those repositories to install them. Make sure to have xformers installed. You probably also need to install the NVIDIA toolkit.
2. Prepare training data: You will also need to prepare 5-30 high-quality images of the person you want to teach your embedding. It's better to have at least half of the images as high-quality face pictures. Follow guide above (preprocess images section) to prepare good quality training data.
   2.1. Crop them to 512x512 (you can use a service like brime to do it in bulk) and rename them if needed.
3. Make captions for images: Run automatic1111 and go to the train tab. You will see the preprocess images tab there, click on it. Set the "source directory," "destination directory," and check "Use BLIP for caption." Click preprocess. It will download the model if you run it for the first time. Check and edit the created captions.
4. Get a training model: It is also better to use models with a style close to the training images. If you are using photos, use realistic models such as realistic vision.
5. Prepare directories: Run the kohya_ss training app. Make sure you are in the "Dreamboth TI" tab. Go to the "tools" tab first. You can create the training directory here. Choose the training and destination prompts and write the training object identifier in the "instance prompt" and the class identifier as "class prompt." Press "prepare training data." Check if the destination directory was updated. Then press "copy info to folders tab." Go to it and input the model path.
6. Set up parameters: Move to the "training parameters" tab. We have to set parameters here.
   - In the "token string" field, input your unique object identifier, and as the init word, use something general like "man" or "woman." 
   - The number of vectors is uncertain, but I used 12. More vectors allow you to store more information about the training data in your embedding. But it will also reduce the number of words you can use in prompt with your keyword.
   We will skip training steps, epoch, learning rate, and train batch size for now.
   - Choose "object template" as "template." Set "save every n epochs" to 5 or 10. I used 10.
   - Don't forget to check the "cache latents" and "cache latents to disk" checkboxes.
   - I didn't use warmup to improve training because I don't know how to use it.
   Now move to the "advanced configuration" section. First, check "use xformers" and save the training state here. In the "sample image config," set "Sample every n epochs" and "Sample prompts." I set "Sample every n epochs" to the same value as "save every n epochs."
   If you are lucky, you will be able to run this training with a "batch size" of 2, which significantly increases training speed. You have to adjust your parameters to match this equality:
   batch size * Gradient accumulate steps (advanced configuration tab) = training images number
So if you are as lucky as me to use 2 in batch size, you will have to set "Gradient accumulate steps" as (training images number/2).
For "learning rate" and "max steps," I am using the following scheme (learning rate: max steps):
0.05:10, 0.02:20, 0.01:60, 0.005:200, 0.002:500, 0.001:3000 (mostly used for faces/characters?)
or
0.005:200, 0.0005:500, 0.00005:800, 0.000005:1000 (mostly used for styles?)
Each of them seems to work, and you can experiment with them. You can continue training with lower learning rates to improve results. I got my first results after using the following scheme: 0.05:10, 0.02:20, 0.01:60, 0.005:200, 0.0005:500. I managed to get a recognizable face using the first saved checkpoint in the last training.
7. Start training: You will have to adjust the learning rate and max steps after every training iteration to match the scheme. Remember to update the "Resume from saved training state" field in the "advanced configuration." You can start checking the created embeddings in automatic1111 after about 800-1000 steps to control overtraining. You will probably be able to get good enough results before you finish training. If needed, you can continue training with a low learning rate to improve results.

You can also train lora using kohya_ss with this setup and get results faster if you already have reference images to train lora.
Please note that I am not an expert in this field and I am only beginning to delve into stable diffusion. The guide provided here is based on my research and understanding so far. I still have some questions on how to improve training speed/quality and if it is possible to use warmup to save some time on restarting training. So, I would be glad to hear any suggestions on that. I would also like to get some information about loss and how this value depends on training.
