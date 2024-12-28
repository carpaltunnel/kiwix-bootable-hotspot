# raspbian-kiwix-hotspot
Build process for a custom Raspbian image that includes Kiwix, configured as a hotspot, and automatically starts Kiwix-Serve on boot.

## Environment
This doc covers building Raspbian using [pi-gen](https://github.com/RPi-Distro/pi-gen)

## Steps
1. Update Docker to v27
2. Clone [pi-gen](https://github.com/RPi-Distro/pi-gen) and `cd` to the directory
3. Install `qemu-user-static`
4. Skip stages 3 through 5 to only build the headless/lite version
   1. `touch stage2/SKIP_IMAGES stage2/SKIP_NOOBS`
   2. `touch ./stage3/SKIP ./stage4/SKIP ./stage5/SKIP`
   3. `touch ./stage4/SKIP_IMAGES ./stage5/SKIP_IMAGES`
5. Copy `stage2-kiwix` from this repo into the `pi-gen` folder
6. `./build-docker.sh` to generate image in the `./deploy` directory


## Customization (NOT TESTED YET)
### Disable default "pi" user
Edit `stage1/01-sys-tweaks/00-run.sh` to create users with generated passwords

```bash
https://kmdouglass.github.io/posts/create-a-custom-raspbian-image-with-pi-gen-part-1/
```

### Set locale
Edit `stage0/01-locale/00-debconf`, replace `en_GB.UTF-8` with `en_US.UTF-8`

??? Edit `stage2/01-sys-tweaks/00-debconf`, set timezone(??) and keyboard layout


## Network Setup
