---
layout: post
title:  "Nvidia Cuda 8, cuDNN 6,Tensorflow, and Pytorch on Linux (Ubuntu)"
date:   2018-10-28
description:  "Fast explanation how to install Cuda and cuDNN for TF and Pytorch on Ubuntu (Linux generally)"
categories: posts
author: "Ayoub"
tag: HowTo
---


While trying to configure Tensorflow (GPU version) in Ubuntu 16.04 (ASUS Laptop) to use NVIDIA 950M, I encountred many errors. Here I present the solutions I've found, hoping it might save some time for other Deep Learning fans.
<h3>1- The Ingredient:</h3>
I prefer using the runfile installation. For cuDNN, it seems there's only deb packages, so be it. You can also try the network mode, but from experience, the OS decides to update things on its own and can generate some conflicts. To make sure everything works, we take more authority and we keep it offline.
<ol>
 	<li>NVIDIA Drivers : <a href="http://www.nvidia.com/Download/">http://www.nvidia.com/Download/</a></li>
 	<li>CUDA 8 : <a href="https://developer.nvidia.com/cuda-80-ga2-download-archive">https://developer.nvidia.com/cuda-80-ga2-download-archive</a></li>
 	<li>cuDNN 6 : You'll need to join the NVIDIA developer community <a href="https://developer.nvidia.com/cudnn">https://developer.nvidia.com/cudnn</a></li>
</ol>
<h3>2- Versions details:</h3>
<ol>
 	<li>The current Tensorflow (v1.4 ) supports CUDA 8 and not CUDA 9. Apparently, CUDA 8 has also a patch version, so you'll need to download both files.</li>
 	<li>For cuDNN, I used 6, since it works. You'll only need the runtime version to run Tensorflow.</li>
</ol>
<h3>3- Installation guide:</h3>
<ol>
 	<li>Put all the downloaded packages in the same folder. You'll end up with
<pre class="lang:sh decode:true ">NVIDIA-Linux-x86_64-&lt;version&gt;.run
cuda_8.0.61_375.26_linux.run
cuda_8.0.61.2_linux.run
libcudnn6_6.0.21-1+cuda8.0_amd64.deb</pre>
</li>
 	<li>Remove any previous NVIDIA package :
<pre class="lang:sh decode:true ">sudo apt purge nvidia*
sudo apt autoremove</pre>
</li>
 	<li>Remove any X11 config files:
<pre class="lang:sh decode:true ">sudo rm /etc/X11/xorg.conf</pre>
</li>
 	<li>Create the file etc/modprobe.d/blacklist-nouveau.conf :
<pre class="lang:sh decode:true ">sudo gedit etc/modprobe.d/blacklist-nouveau.conf</pre>add :<pre class="lang:sh decode:true ">blacklist nouveau
options nouveau modeset=0</pre>
</li>
 	<li>Update the headers and memorize 7 then restart :
<pre class="lang:sh decode:true ">sudo update-initramfs</pre>
</li>
 	<li>Once in the login screen, press Alt+Ctrl+F1 to move to terminal mode, if you don't have login screen, do 8 and then 7</li>
 	<li>Turn off the lights and remember how to turn them on (replace stop by start):
<pre class="lang:sh decode:true ">sudo service lightdm stop</pre>
</li>
 	<li>Make sure you're in the drivers folder</li>
 	<li>Install the drivers, with the No OpenGL option !!! : 'No OpenGL' implies that the GPU will not be used for display if integrated graphics are availables
<pre class="lang:sh decode:true ">sudo ./NVIDIA-Linux-x86_64-384.111.run --no-opengl-files</pre>
</li>
 	<li>Turn on the lights</li>
 	<li>Check drivers are installed :
<pre class="lang:sh decode:true ">cat /proc/driver/nvidia/version
	&gt; NVRM version: NVIDIA UNIX x86_64 Kernel Module 384.111 Tue Dec 19 23:51:45 PST 2017
	&gt; GCC version: gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.5)</pre>
</li>
 	<li>Install CUDA 8 without the NVIDIA Drivers and keep the default folders:
<pre class="lang:sh decode:true ">sudo ./cuda_8.0.61_375.26_linux.run</pre>
</li>
 	<li>Install CUDA 8 Patch :
<pre class="lang:sh decode:true ">sudo ./cuda_8.0.61.2_linux.run</pre>
</li>
 	<li>CUDA will add a symbolic link /usr/local/cuda to point to /usr/local/cuda-8-0, so we will just add the link to environment paths in the file ~/.bashrc
<pre class="lang:sh decode:true ">gedit ~/.bashrc</pre> and add the following lines<pre class="lang:sh decode:true ">export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH</pre>Then either restart or load the changes using :<pre class="lang:sh decode:true ">source ~/.bashrc</pre>
</li>
 	<li>Check CUDA is installed :
<pre class="lang:sh decode:true ">nvcc -V</pre>
</li>
 	<li>Install cuDNN 6 for CUDA 8 libraries :
<pre class="lang:sh decode:true ">sudo dpkg -i libcudnn6_6.0.21-1+cuda8.0_amd64.deb</pre>
</li>
 	<li>Install tensorflow gpu :
<pre class="lang:sh decode:true ">pip3 install tensorflow-gpu</pre>
</li>
</ol>
<h3>4- Troubleshooting</h3>
Now you can test tensorflow with this small program:
<pre class="lang:python decode:true ">import tensorflow as tf
with tf.Session() as session:
	b = tf.constant(1.0)
	print('b = %f'%session.run(b))</pre>
If everything was done correctly, you should get a long output from CUDA and tensorflow followed by the result :
<pre class="lang:default decode:true">b = 1.000000</pre>
Sometimes you encounter a problem in tensorflow code and end up aborting the computation without setting the GPU VRAM free. In this case, run:
<pre class="lang:sh decode:true ">nvidia-smi</pre>
and you'll get something like this:
<img class=" wp-image-54 aligncenter" src="https://ayoub.ghriss.net/wp-content/uploads/2018/01/Nvidia_smi-300x180.png" alt="" width="374" height="224" />
Then all you have to is to kill the process:
<pre class="lang:sh decode:true ">kill -9 3193</pre>
