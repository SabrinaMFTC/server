# Instructions on how to setup your own server

## Summary

1. [Setting up your server and hardware](#1-setting-up-your-server-and-hardware)

---

# 1. Setting up your server and hardware

**1.** Setup your server

- In our case, we used:
  - Two Raspberry Pi 4 Model B
  - Two SSD's 128 GB
  - One cooler
  - Pi case / rig

**2.** Buy a domain (you can use [Namecheap](https://www.namecheap.com/), for example).

**3.** Create a [Cloudflare](https://www.cloudflare.com/) account and set Cloudflare as your DNS provider on Namecheap.

- Wait until DNS challenge is completed.

**4.** Install Raspberry Pi Imager

```bash
sudo apt update
sudo apt install rpi-imager
```

**5.** Configure it as follows:

- `Choose Device`: `Raspberry Pi 4`
- `Choose OS`: `Raspberry Pi > Raspberry Pi OS (other) > Raspberry Pi OS Lite (64-bit)`
- `Choose Storage`: select the SSD's

**6.** Configure the nodes - in our case, we will have two: a `manager` or `headnode` and a `worker`.

**7.** Configure the WiFi connection and allow SSH access.

---

# 2.

**8.**
