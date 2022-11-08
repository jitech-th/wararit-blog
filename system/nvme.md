<img width="650" alt="image" src="https://user-images.githubusercontent.com/31476202/200504876-c629478c-552c-421c-917f-474bebb51183.png">

Find the line that starts with GRUB_CMDLINE_LINUX_DEFAULT and add the parameter there. Your line should look like this now (Make sure it is a single line with no line break)

`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvme_core.default_ps_max_latency_us=5500"`

And update grub using

`sudo update-grub`
