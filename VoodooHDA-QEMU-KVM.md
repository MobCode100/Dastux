# Audio Playback Without Passthrough in macOS QEMU KVM using VoodooHDA

[OneClick macOS Simple KVM](https://oneclick-macos-simple-kvm.notaperson535.is-a.dev/), [OSX-KVM](https://oneclick-macos-simple-kvm.notaperson535.is-a.dev/), [Docker-OSX](https://github.com/sickcodes/Docker-OSX), and [Quickemu](https://github.com/quickemu-project/quickemu) are all great projects that allow users to run macOS/Hackintosh virtually in virtual machine. However, the playing audio will not work out of the box. To my surprise, there aren't many tutorial/guide on how to install VoodooHDA in macOS VM, and is why I've decided to do one. This guide is expected to work on Big Sur, Monterey and Ventura. Older versions might be problematic (testing needed).

## Is it worth it?
Depends on your use case. All those projects recommended passing through audio card for the reason of better experience than using VoodooHDA. VoodooHDA has a poor audio quality, you will most likely hear stuttering or distortion. Most likely you'll not find an enjoyable experience, this is more for convenience rather than entertainment purpose. I don't found any better way other than having to passthrough audio device.

## Getting started
A virtual audio output device must be properly configured by simply passing some arguments to `qemu-system-x86_64` so macOS detects it and able to use VoodooHDA as its driver. Something like 
```
-audiodev pa,id=audio0 -device intel-hda -device hda-output,audiodev=audio0
```
In this example, QEMU will transfer the audio to your PulseAudio backend denoted by `pa`. The command is usually located inside a script file (_OSX-KVM_: OpenCore-Boot.sh, _OmSK_: basic.sh, _Quickemu_: macos-big-sur.sh). Please read documentations for the best way to achieve this.

Next, you have to make sure you are able to boot into macOS recovery partition (see below). In Docker-OSX, the partition can be enabled using `-e NOPICKER=false` at container creation. 

## Installing VoodooHDA

### Disable System Integrity Protection (SIP)
SIP is a security mechanism that must be disabled, or else you might get a "bad signature" error.
Select the recovery partition (arrow keys) and press enter to boot into it.

![Recovery partition in OpenCore](/images/Recovery_in_OpenCore.png)

Go to Utilities > Terminal, enter the command `csrutil status`. If you see either the line `System Integrity Protection status: disabled` or `Kext Signing: disabled`, then you can skip this step, restart and proceed to [download the latest kernel extension (kext)](#download-latest-voodoohda)

Run the below command (choose either one)
```
csrutil disable
# or
csrutil enable --without kext
```
Reboot into macOS.
### Download latest VoodooHDA
Once you have booted into your macOS, open Safari and download latest VoodooHDA kext from this [site](https://sourceforge.net/projects/voodoohda).

Open the terminal application, run `sudo cp -R Downloads/VoodooHDA.kext /Library/Extensions`, enter password when prompted. This will copy the kext to the extensions folder. Replace the `Downloads` to the path of the kext if you download it in a different folder.

Shortly after that, a popup will appear.

![System Extension Updated](/images/Kext_Popup.png)

For Ventura, you might need to load the kext manually by entering `sudo kextutil -v /Library/Extensions/VoodooHDA.kext` for the popup message to appear.

### Enable the extension
Go to Security & Privacy section inside System Settings application or you can just click the **Open Security Preferences** button.

![Security & Privacy](/images/Security_%26_Privacy_macOS.png)

Click on the lock, enter your password, click allow and restart.

The UI is different for Ventura, but you can easily find 'VoodooHDA' in the same section.

### Audio testing
After restarting, if everything is fine you should be able to find the 'Line out (Green N/A)' device. If the audio performance is decent enough for you, then congrats! You have successfully installed VoodooHDA on your macOS/Hackintosh VM.

![Sound Settings](/images/Sound_Settings.png)

## Feedback and contributing 
Got questions, problems, ideas or improvements? Just open a pull request or create an issue, I will try my best to help. I'm looking for better solution than VoodooHDA without having to passthrough audio device, anything would be very appreciated.

## Resources & references
1. [InsanelyMac: VoodooHDA 3.01 (Credit: Slice)](https://www.insanelymac.com/forum/topic/314406-voodoohda-301/?do=findComment&comment=2756841)
2. [GitHub: installVoodooHDA4BSnMont (Credit: yahgoo)](https://github.com/yahgoo/installVoodooHDA4BSnMont)
2. [Apple Developer Forums: Enabling parts of System Integrity Protection while disabling specific parts?](https://developer.apple.com/forums/thread/17452)