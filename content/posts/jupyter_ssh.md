+++
title = "Jupyter notebooks over SSH for remote deep learning on a GPU"
author = ["Ryan Cummings"]
date = 2020-02-28T20:09:00-05:00
lastmod = 2020-02-28T21:19:18-05:00
tags = ["python", "ai"]
categories = ["technical"]
draft = false
+++

[<img src="/img/misc/jupyter_logo.png" alt="jupyter_logo.png" width="200" />](/img/misc/jupyter_logo.png)
I love [Jupyter](https://jupyter.org/) notebooks. As someone who primarily works on data science projects in Python, Jupyter is probably the most important tool in my toolchain. For the uninitiated, Jupyter notebooks are documents that can contain live code, visualizations, and descriptive text. By breaking code into chunks, Jupyter notebooks let you run your code in a non-linear way, jumping from block to block and letting Jupyter keep track of variables and environment. Rather than writing a python file and then running the whole thing, Jupyter lets you add in print() statements and visualization commands half-way through the execution of your code easy.

Youtuber Corey Schafer has an amazing breakdown, shown below:

<iframe width="925" height="529" src="https://www.youtube.com/embed/HW29067qVWk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Remote access to notebook {#remote-access-to-notebook}

I use Jupyter for deep machine learning, taking advantage of a gaming pc with a gpu. This pc runs Ubuntu linux (and dual-boots Windows for actual gaming) and is accessible over my network by SSH. It is possible to work on this computer, but I would much rather work remotely from my laptop and external monitor.

To accomplish this task, I created the following shell script:

```shell
#!/bin/bash
#title           : Jupyter connection
#description     : Connect to jupyter notebook
#author          : Ryan Cummings
#date            : 20190229

ssh -o StrictHostKeyChecking=no gamer "source ~/anaconda3/etc/profile.d/conda.sh; conda activate pytorch; jupyter notebook --no-browser --port=8887" &
urxvt -e sh -c "ssh -N -L localhost:8888:localhost:8887 USERNAME@gamer"
```

This very simple script does two things. The first command logs into my gaming pc via SSH using the "gamer" profile I set up elsewhere in my `.ssh/config` file. It then activates my Anaconda virtual python environment, which is necessary to run the jupyter notebook. Lastly, it runs `jupyter notebook` on port 8887 with no browser.

The second command (which opens without waiting for the first to finish, because of the `&` symbol) opens a new urxvt terminal window and opens a tunnel to my gaming pc. The tunnel links my laptop's port 8888 with the gaming pc's port 8887, where the jupyter notebook is running. The terminal will look blank, but the port should be ready.

Now, all you have to do is click the url provided in the first terminal window where the jupyter notebook is running. A web browser window will open pointing to localhost:8887/LONG\_TOKEN\_TEXT. Change the 7 to an 8 and you will login to the notebook with the token in the long URL. And that's it! Any code you run will run on the remote notebook server, and you can use the GPU to your heart's content.

It took me a little while to figure this out. I hope it helps out another data scientist in need!
