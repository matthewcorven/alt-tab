---
title: Nerd Husband - Honey-do Good Deed
tags: nerd husband raspberrypi point-of-sale
---

We nerd husbands are always looking for ways to make our wives' lives easier.

I've been working on a point of sale system for my wife's business.  She had been wanting a digital display to give her dog grooming clients a personal and professional touch, displaying short autobiographies of her staff intermingled with reminders to book ahead; Future suggested booking dates were also to be displayed as quick reference, calculated automatically based on awareness of days the business is scheduled to be closed.  The display was to be mounted on the wall, and the content was to be updated remotely and as self-sufficent as possible, right down to the daily updates of the screen itself.

## Phase 1: Idle thought, minimal progress

Guilty as charged.  It took a few months to stop pinballing ideas and throwing them away just as quickly.  In any case by the end of it all I was clear on the hardware.  I had previously made Raspberry Pi boards a part of the home that we all rely on, such as the pihole that has dutifully provided amazing advertising blocking for the entire home network for years.  So I was familiar with the hardware over a few generations of Pis, and comfortable with the idea of using one for this project.  Like any other self-respecting nerd-dad with no time, I had an unused Raspberry Pi 4 B from another project that never got off the ground.  For the display I selected an inexpensive 12" external LCD, ensuring that it supported VESA mounting (which turned out to be uncommon).

Aside from mounting and running cables, the hardware was pretty much ready to go.  I had a few ideas for the software, and went on to try a handful of homebrew approaches before leaning into an available open-source option.

## Phase 2:  How will employees manage the scheduled closing dates?

## Phase 4: Approach challenge - How do I automate the operational steps of a user-facing template having inline "function" syntax for our own customizations

## Phase 5: Approach challenge - Where will I store configuration & content data?  How will it be used by owner versus the role of specific employees?

## Phase 6: Approach challenge - How do I minimize the need for servicing the hardware as well as the software?


## Phase 7: Approach challenge - How do I maximize the display's uptime?

### Phase 7.1: Micro-SD card wear

#### Setting up a Read-Only Filesystem

SD cards have a limited number of write cycles, and the Raspberry Pi's operating system is constantly writing to the SD card.  This is a problem because the SD card is the only storage on the Raspberry Pi, and if it fails the Raspberry Pi will no longer boot.  The solution is to use a read-only filesystem, which prevents writes to the SD card.

Avoiding writes to the disk entirely is the best way to prevent SD card wear, although it requires configuration changes and a willingness to accept some limitations.  Chief among these limitations is that the filesystem cannot be written to, so any changes you make to the filesystem will be lost when the Raspberry Pi is rebooted.  RAM can provide an area for writes during normal use as performed by system and user apps, but all those changes will be purged.  In my case, I don't need to store anything after final setup of the device, so this is not a problem and I'm happy to accept the limitations for the sake of the SD card's longevity.

Using [this Adafruit guide](https://learn.adafruit.com/read-only-raspberry-pi/overview), I set up a read-only filesystem.  Note that it's easy to temporarily disable the read-only filesystem by remounting it as read-write, so I can still make changes to the filesystem when needed.

### Phase 7.2: Micro-SD card loss

There are roughly 14,495 different things that can go wrong with an SD card, at least by my [estimation](https://media.giphy.com/media/l2QEaGSBrzAE5tICQ/giphy.gif).  Even with a read-only filesystem, the SD card is still required to boot the Raspberry Pi.  If the SD card fails, the Raspberry Pi will no longer boot.  My solution is to have backups, exact bit-for-bit clones, with which I can swoop in and save the day with.  On Windows, I use [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/) to create the backup image, which I then write to backup cards using [etcher](https://www.balena.io/etcher/). Each card is labeled with a handheld thermal label maker with plastic tape, [dropped in a water-proof sealed container and finally stored in a fire-resistant safe](https://media.giphy.com/media/QUENDfi6DEMLzQ0CKt/giphy.gif).

### Phase 7.3: Power loss or fluctuations

If power is lost, this display going black is the least of concerns for the business owner.  I'm more concerned about the longevity of the hardware, so I want to buffer it from power fluctuations.  My standard solution is to use a UPS (uninterruptible power supply) to provide a stable power source, and to use a script to monitor the power level and shut down the Raspberry Pi when the power level is low.  This requires the UPS to be connected to the Raspberry Pi via USB... but what about the two desktop computers elsewhere in the groom shop?  Until now I hadn't considered the need for a UPS for them, but it's a good idea.

So it seems there's much more to do here, and I may as well focus on it as a separate matter.  For now, I'll assume the power is stable and the Raspberry Pi will not lose power.  I'll revisit this later.  For now, I can still rest assured that the read-only filesystem will *help* prevent the Raspberry Pi's SD card from being corrupted by a sudden power loss.

### Phase 7.4: Wake-up Race Condition

If the Raspberry Pi wakes up before the display, the Pi will not be able to read the display's EDID (Extended Display Identification Data).  Without an awareness of which configurations your display supports, the Pi will make a guess that may be right or may render the display distorted or inoperable.  One suggestion I read described manually setting the `hdmi_group` and `hdmi_mode` properties of `config.txt` to match the display's EDID data.  This is a good idea, but I'd rather not have to do this when replacing a display or adding additional installations.  I'd rather have the Pi read the EDID data once it has opportunity, and at that time automatically set the `hdmi_group` and `hdmi_mode` properties for subsequent boots.  I found this mentioned in [this post](https://www.raspberrypi.org/forums/viewtopic.php?t=245931).  The solution is to add the following line to `/boot/config.txt`:

```
hdmi_force_hotplug=1
```

This forces the Pi to read the display's EDID data and set the `hdmi_group` and `hdmi_mode` properties automatically.  This is a good solution, but it's not perfect.  The Pi will still boot before the display, and the Pi will still not be able to read the display's EDID data.  So the Pi will still make a guess, and it may be right or it may render the display distorted or inoperable.  But the Pi will *eventually* read the display's EDID data and set the `hdmi_group` and `hdmi_mode` properties automatically.  So the display will eventually be correct, but it may take a few minutes.  This is not ideal, but it's good enough for now.

### Phase 7.4: Display failure

The display is relatively cheap, just one of many available on Amazon as an "external USB display".  I don't have a spare, but they are plentiful.  If the display fails, I can replace it quickly and easily.

### Phase 7.5: Network failure

The network and internet connection is required for the Raspberry Pi to view as well as update its content.  So if the Pi ever boots (or reboots) while the connection is down, there will be no content to display.  My solution is to maintain a backup of the content on a separate external USB drive, which is *NOT* read-only like the root filesystem.  So each time content changes, I'll ensure to store that data on the USB drive.  If the network fails, the Pi will continue to display the content from the USB drive.  When the network is restored, the Pi will update its content from the internet and also update the USB drive again.

### Phase 7.6: Software failure

### Phase 7.7: Hardware failure

## Phase 8: Approach challenge - How do I minimize the cost of the hardware?

## Phase 9: Approach challenge - How do I squeeze out performance?

sudo service --status-all
sudo service cups stop
sudo service cups-browsed stop
sudo service bluetooth stop
sudo systemctl disable cups
sudo systemctl disable cups-browsed
sudo systemctl disable bluetooth

sudo apt remove --purge cups cups-browsed bluetooth

sudo apt remove pulseaudio pulseaudio-utils pulseaudio-module-bluetooth



## Phase 10: Approach challenge - How do I minimize the cost of the content?

## Phase 11: Approach challenge - How do I minimize the cost of the configuration?

## Phase 12: Approach challenge - How do I minimize the cost of the maintenance?

## Phase 13: Approach challenge - How do I minimize the cost of the support?

## Phase 14: Approach challenge - How do I minimize the cost of the training?

## Phase 15: Approach challenge - How do I minimize the cost of the development?


## Setup Process (Provisioning a new master image)

1. Download and install Raspberry Pi Imager.  
2. Once open, select "Choose OS" and select "Raspberry Pi OS (64-bit)".  
3. Select "Choose SD Card" and select the SD card.  
4. Open the cog icon and set your desired options.  My settings:
   1. Hostname = "grooming-display"
   2. SSH enabled.  This will allow me to connect to the Raspberry Pi remotely using SSH.  Not only is still essential during development and deployment in the engineering space, it will also be useful for onsite troubleshooting and maintenance.
5. Set username and password explicitly.  This is essential for security.  The default username and password are well-known and published online, so it's important to change them.
6. Set SSID & password for WiFi.  I want to be able to connect to the Raspberry Pi remotely using SSH, so I need to connect it to the network.
7. Set timezone and keyboard layout.
8. Select "Write".  
9. Once complete, eject re-insert the SD card.  This will allow you to access the boot partition, to change some configuration.

## Configuration

1. Open the boot partition (`/boot`).  This is the only partition that is accessible from Windows.
2. Open `config.txt` in a plain text editor.
3. Disable bluetooth and audio, as well as camera detection:
    1. Add `dtoverlay=disable-bt` to the bottom of the file
    2. Add `dtparam=audio=off`
    3. Add `camera_auto_detect=0`
    4. Add `hdmi_blanking=1` (display will be told to turn **off** when Raspberry Pi is off or tells the display to "sleep")
    5. Set `hdmi_force_hotplug=1` (keep HDMI port active even if no display is detected *yet*)
 
## Pre-boot checklist

1. Connect display to the mini HDMI port ([need an adapter?](https://www.amazon.com/s?k=mini+hdmi+to+hdmi+adapter&sprefix=mini+hdmi+to+%2Caps%2C104&ref=nb_sb_ss_ts-doa-p_3_13)).
2. Connect keyboard and mouse. 
3. Connect the "power" USB-C to an approved power supply ([here](https://www.raspberrypi.com/products/type-c-power-supply/), [here](https://www.adafruit.com/product/4298) or [here](https://www.amazon.com/CanaKit-Raspberry-Power-Supply-USB-C/dp/B07TYQRXTK))

## First Boot

This takes longer that what a normal boot time will be.  In this first run you'll see actions running such as resizing the filesystem to the full size of the SD card, and generating SSH keys.  Eventually, you'll see the login prompt.
```bash
storedisplay login: screen
Password: [YOUR_PASSWORD]
```

Once logged in:
1. `mkdir ~/temp`
1. Update the system:
```bash
sudo apt update
sudo apt upgrade
```
2. Install and setup Samba
```bash
mkdir ~/display_share
sudo apt install samba -y
sudo nano /etc/samba/smb.conf
```
...inside the file:
```bash
[display_share]
path = /home/screen/display_share
browsable = yes
writeable = yes
create mask = 0666
directory mask = 0777
public = yes
read only = no
guest ok = yes
```
3. Make the share folder writable by anyone:
```bash
sudo chmod 0777 /home/screen/display_share
```
4. Restart Samba
```bash
sudo systemctl restart smbd
# or
sudo service smbd restart
```
5. A few desktop configuration changes:
- Settings > Appearance
  - Desktop
    - Set wallpaper, color
    - Disable desktop icons
  - Taskbar
    - Set size to small
1. Disable screen blanking (display disabling as a power saving feature).  Do this in LightDM rather than LXDE, so that it applies to all users and also earlier in the boot process.
```bash
sudo nano /etc/lightdm/lightdm.conf
```
...inside the file, edit the `[Seat:*]` section to edit or add the following line:
```bash
xserver-command=X -s 0 dpms
```
1. Disable screen saver
```bash
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```
...inside the file:
```bash
@xscreensaver -no-splash
```
1. Setup UFW (Uncomplicated Firewall) and open a port for the REST API
```bash
sudo apt install ufw
sudo ufw disable
sudo systemctl stop ufw.service
sudo ufw allow ssh
sudo ufw allow 2600/tcp comment 'accept HTTP traffic to Woof REST API'
sudo systemctl start ufw.service
sudo systemctl status ufw.service
sudo ufw enable
sudo ufw reload
```

1. Install dotnet SDK
```bash
wget https://download.visualstudio.microsoft.com/download/pr/93db1aea-6913-4cdc-8129-23e3e3de8dd1/4a942a2fbbb6ca6667c01ec414096ee0/dotnet-sdk-8.0.100-preview.5.23303.2-linux-arm64.tar.gz
mkdir -p $HOME/dotnet && tar zxf dotnet-sdk-8.0.100-preview.5.23303.2-linux-arm64.tar.gz -C $HOME/dotnet

export DOTNET_ROOT=$HOME/dotnet
export PATH=$PATH:$HOME/dotnet
export DISPLAY=:0

echo 'export DOTNET_ROOT=$HOME/dotnet' >> ~/.bashrc
echo 'export PATH=$PATH:$HOME/dotnet' >> ~/.bashrc
echo 'export DOTNET_CLI_TELEMETRY_OPTOUT=1' >> ~/.bashrc
echo 'export DISPLAY=:0' >> ~/.bashrc
echo 'export FONTCONFIG_PATH=/etc/fonts' >> ~/.bashrc

sudo ln -s ~/dotnet/dotnet /usr/bin/dotnet

```
1. Set up a certification for HTTPS
```bash
sudo apt install libnss3-tools openssl

# Create a private key and certificate for the CA:
openssl genpkey -algorithm RSA -out ca.key
openssl req -x509 -new -nodes -key ca.key -days 365 -out ca.crt

# Create a private key for the server:
openssl genpkey -algorithm RSA -out server.key

# Create a certificate signing request (CSR) for the server:
openssl req -new -key server.key -out server.csr
# ... You will be prompted to enter details for the server certificate. For 'Common Name' enter the hostname of the Raspberry Pi (storedisplay.local)

# Use the CA to sign the server's CSR, generating the server's certificate:
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365

# Finally, package the server's private key and certificate into a PFX file:
openssl pkcs12 -export -out server.pfx -inkey server.key -in server.crt -certfile ca.crt
#  ... You will be prompted to enter a password. This password is required to import the PFX file later.

# Add to Raspberry Pi's trusted certificates
sudo mkdir -p /usr/local/share/ca-certificates/woof
sudo cp ca.crt /usr/local/share/ca-certificates/woof/woof.crt
sudo update-ca-certificates

# For Chromium/Chrome/Edge on Ubuntu
sudo certutil -d sql:$HOME/.pki/nssdb -A -t "P,," -n localhost -i /usr/local/share/ca-certificates/woof/woof.crt
sudo certutil -d sql:$HOME/.pki/nssdb -A -t "C,," -n localhost -i /usr/local/share/ca-certificates/woof/woof.crt

# For Firefox on Ubuntu
sudo mkdir /usr/lib/firefox
sudo mkdir /usr/lib/firefox/distribution
cat <<EOF | sudo tee /usr/lib/firefox/distribution/policies.json
{
    "policies": {
        "Certificates": {
            "Install": [
                "/usr/local/share/ca-certificates/woof/woof.crt"
            ]
        }
    }
}
EOF
```

1. Setup an SSH key
```bash
ssh-keygen -t rsa -b 4096 -C "[YOUR_GITHUB_REGISTERED_EMAIL_ADDRESS]"
# Ensure ssh-agent is Running
eval "$(ssh-agent -s)"
# Display the public key (copy this)
cat ~/.ssh/id_rsa.pub
```
1. Add the key to GitHub:
  - Go to [GitHub](https://github.com) and login.
  - Click on your profile picture in the top right corner and select "Settings".
  - Click on "SSH and GPG Keys" in the left menu.
  - Click "New SSH key"
  - For "Title", enter "grooming-display"
  - For "Key type", select "Authentication Key"
  - For "Key", paste the public key you copied earlier.
  - Click "Add SSH key" to submit form

1. Clone Woof repo
```bash
mkdir ~/git
cd ~/git
git clone git@github.com:[GITHUB_USERNAME]/[REPOSITORY_NAME].git
```

1. Prepare VSCode for remotely connecting to the Raspberry Pi, which will act as the active development host, build server, as well as the deployment target.
  - Open VSCode
  - Install the [Remote SSH extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)
  - Install the C# extension










============================
https://www.cyberciti.biz/faq/how-to-configure-firewall-with-ufw-on-ubuntu-20-04-lts/
https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/?tabs=net7
https://www.chromium.org/developers/how-tos/run-chromium-with-flags/
https://www.labnol.org/internet/direct-links-for-google-drive/28356/
