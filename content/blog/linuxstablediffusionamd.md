---
author: Thomas Cross
title: Installing Stable Diffusion Web-UI on Arch-based systems with AMD GPUs
date: 2022-10-15
description: Utilizing ROCm on an Arch Distro to run Stable-Diffusion.
tags: ["stable-diffusion", "ROCm", "AI", "Image Generation"]
thumbnail:
  url: blog/AMDrx6000.jpeg
---

# Introduction:

This guide was written with Arch-Linux based systems in mind. I am using Manjaro 21.3.1 Ruah for this guide.

I will primarily be following the guide presented on the stable-diffusion webui page, though with some added tweaks to avoid having to root around for answers like I had to.

---

# Getting Started:

You will need the following:

- Arch Based distro or Arch Linux itself.
- AMD GPU (I am using an RX 6700 non-xt, but this should work for others.)
- ROCm kernel drivers (This is substantially easier on Arch thanks to the AUR)
- The newest version of stable-diffusion-webui
- Docker (This can also work without it, so I'd suggest trying it without docker first and if it doesn't work, try it with docker.)

---

# Installing ROCm Kernel

ROCm is the platform you will have to use to pass the PyTorch CUDA check. To install, make sure you've enabled AUR support in your software manager. You can also enable this on the CLI with the command:
    `sudo sed -Ei '/EnableAUR/s/^#//' /etc/pamac.conf`
You want to install the rocm-opencl-runtime & rtocminfo from the AUR afterwards.
You can do this graphically in the included package manager if your arch-based distro supports it or by using the command:

    sudo pamac search rocm
    sudo pamac install rocm-opencl-runtime
    sudo pamac install rocm-info

Keep in mind that ROCm officially supported on LTS releases of operating systems, so this can/will be easier on certain systems.
You will have to compile ROCm for yourself if your system isn't supported.

---

# Installing Docker (Optional)

Go to your software manager and install the offical repository version, or by using the command

`sudo -S docker`

After installation, you will need to enable docker by typing

`sudo systemctl start docker`

Then, pull the latest version of pytorch

`docker pull rocm/pytorch`

Afterward, use the following commands:

`alias drun='sudo docker run -it --network=host --device=/dev/kfd --device=/dev/dri --group-add=video --ipc=host --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -v $HOME/dockerx:/dockerx'`

`drun rocm/pytorch​`

Note that you may have to use a different version of Pytorch if you are using a Navi 21 GPU and based on your operating system. However, you probably won't have to do this, but if you do, please take a look at the tags section on the dockerfile.

---

# Installing Stable Diffusion (Docker)

Use the commands

    cd /dockerx
    git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui

If that directory doesn't exist, create it with

    mkdir dockerx
    cd stable-diffusion-webui

Use python --version to check the version of python you're running. It's best practice to update python inside of the container with the following commands:

    apt install python3.X-full # Confirm every prompt #Use whichever version is suggested
    update-alternatives --install /usr/local/bin/python python /usr/bin/python3.X 1
    echo 'PATH=/usr/local/bin:$PATH' >> ~/.bashrc

Restart the container after the fact. If you're doing this after you've created the virtual environment, remove and then recreate the venv.

    python -m venv venv
    source venv/bin/activate
    python -m pip install --upgrade pip wheel

---

# Installing Stable Diffusion (Normally)

If you're just installing to a regular folder, create a directory and then use the following commands in the terminal. It's almost exactly the same as installing with docker.

    git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
    cd stable-diffusion-webui
    python -m venv venv
    source venv/bin/activate
    python -m pip install --upgrade pip wheel
---
# Running Stable Diffusion (Both)

The main guide for stable diffusion often requires you to run Stable Diffusion with their torch commands. However, there's a good reason why that doesn't work.

If you run it as is, it will return an error about not passing the cuda check. DO NOT use --skip-torch-cuda-test in the args. This will allow the program to run, yes, but it will either cause another error down the road or not use the GPU, which is critical in returning images of good quality at a reasonable amount of time. Therefore, you absolutely have to run some derivative of this:

`COMMAND='pip install torch torchvision --extra-index-url https://download.pytorch.org/whl/rocm5.1.1' HSA_OVERRIDE_GFX_VERSION=10.3.0 REQS_FILE='requirements.txt' python launch.py --precision full --no-half HSA_OVERRIDE_GFX_VERSION`

This essentially tells Pytorch that you should recognize this graphics driver. Note that you may need to use a different version number. Furthermore, the attachments on the end can vary, but --precision full and --no-half are essential in keeping s/it (seconds per iteration) low.

---

# Final Notes:

For GPUs with less than 10GBs of VRAM, you should use --medvram for GPUs with 6-8GBs, and --lowvram for GPUs with less than 6GB. --medvram is always good to use even on higher end GPUs since it can minimize VRAM constraints as well. for 8GBs or lower, you should use --precision half instead.

This will take quite a while as it sets up for the first time, but once it's done, you should have a functioning environment that you can test with appropriate models.

I'd strongly suggest following this guide: https://rentry.org/sdg_FAQ installing the novelai leak, though you're free to use the same guide to install other models that you might be interested in using.

For each successive run, you'll need to reinitialize the virtual environment and git pull the latest version of the repository.

Hopefully, that clears anything up for any AMD GPU users. This method works perfectly on my card, as it generates a 512x512 image with about 1.3 seconds per iteration.

---