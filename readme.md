This script lets you set a custom GPU fan curve on a headless Linux server.

```text
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 430.40       Driver Version: 430.40       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce RTX 208...  On   | 00000000:08:00.0 Off |                  N/A |
| 75%   60C    P2   254W / 250W |   9560MiB / 11019MiB |    100%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GeForce RTX 208...  On   | 00000000:41:00.0  On |                  N/A |
| 90%   70C    P2   237W / 250W |   9556MiB / 11016MiB |     99%      Default |
+-------------------------------+----------------------+----------------------+
```

### Instructions
Copy the `fans.py` script into a local directory and run
```
python fans.py --speed 99 99 
``` 
If you hear your server take off, it works! Now interrupt it and re-run either with Sensible Defaults (TM),
```
python fans.py
```
or you can pass your own parameters with 
```
python fans.py --temp 17 84 --speed 15 99 
```
This will make the fan speed increase linearly from 15% at 17C to 99% at 84C.  You can also increase `--hyst` if you want to smooth out oscillations, at the cost of the fans possibly going faster than they need to.

### Why's this necessary?
If you want to install multiple GPUs in a single machine, you have to use blower-style GPUs else the hot exhaust builds up in your case. Blower-style GPUs can get _very loud_, so to avoid annoying customers nvidia artifically limits their fans to ~50% duty. At 50% duty and a heavy workload, blower-style GPUs will hot up to 85C or so and throttle themselves. 

Now if you're on Windows nvidia happily lets you override that limit by setting a custom fan curve. If you're on Linux though you need to use `nvidia-settings`, which - as of Sept 2019 - requires a display attached to each GPU you want to set the fan for. This is a pain to set up, as is checking the GPU temp every few seconds and adjusting the fan speed. 

This script does all that for you.

### How it works
When you run `fans.py`, it sets up a temporary X server for each GPU with a fake display attached. Then, it loops over the GPUs every few seconds and sets the fan speed according to their temperature. When the script dies, it returns control of the fans to the drivers and cleans up the X servers.

### I want a different curve
The fan speeds are calculated from the `TEMPS` and `SPEEDS` arrays in the script; alter them if you'd like something different.

### It doesn't work
Check that you've got `XOrg`, `nvidia-settings` and `nvidia-smi` on the `PATH`. Check you don't have a display attached. Otherwise, add breakpoints and print statements till you figure it out!

### Credit
This is based on [this 2016 script](https://github.com/boris-dimitrov/set_gpu_fans_public) by [Boris Dimitrov](dimiroll@gmail.com), which is in turn based on [this 2011 script](https://sites.google.com/site/akohlmey/random-hacks/nvidia-gpu-coolness) by [Axel Kohlmeyer](akohlmey@gmail.com).
