# Setting up Cloudflare WARP on Chromecast with Google TV

## Introduction
Cloudflare WARP is a free VPN service that can be used to secure your internet connection. 
Unfortunately, Cloudflare WARP does not have a native app for Android TV devices.
This guide will show you how to set up Cloudflare WARP on your Chromecast with Google TV.

## Prerequisites

1. Chromecast with Google TV (Obviously)
2. A computer with [Docker](https://www.docker.com/), [Orbstack](https://orbstack.dev/) or [Podman](https://podman.io/) installed
3. An Android phone with [Send Files to TV](https://sendfilestotv.app/) app installed

## Steps

### 1. Install the necessary apps on your Chromecast

1. Launch the Google Play Store on your Chromecast
2. Search for and install the following apps:
   - [FX File Explorer](https://play.google.com/store/apps/details?id=nextapp.fx)
   - [WireGuard](https://play.google.com/store/apps/details?id=com.wireguard.android)
   - [Send Files to TV](https://play.google.com/store/apps/details?id=com.yablio.sendfilestotv)

### 2. Generate WireGuard configuration on your computer

1. From a terminal run the following command to start a Docker container with the necessary tools:
    ```bash
    docker run --rm -it neilpang/wgcf-docker /bin/bash
    ```
2. Inside the container, run the following command to generate a WireGuard configuration file and print the necessary keys to the terminal:
    ```bash
    yes | wgcf register && wgcf generate && echo -e "\n\n---OUTPUT---" && cat wgcf-profile.conf | grep "Key" && echo -e "------------\n\n" && exit
   ```
3. Copy the `PrivateKey` and `PublicKey` values from the output.
4. Create a new file on your computer called `wg-client.conf` and paste the following content:
    ```conf
    [Interface]
    PrivateKey = <PrivateKey>
    Address = <IPAddress>/32
    DNS = 1.1.1.1, 1.0.0.1
    MTU = 1280
    [Peer]
    PublicKey = <PublicKey>
    AllowedIPs = 0.0.0.0/0
    AllowedIPs = ::/0
    Endpoint = engage.cloudflareclient.com:2408
   ```
5. Replace `<PrivateKey>` and `<PublicKey>` with the values you copied in step 2.3.

### 3. Get the IP address of your Chromecast

1. Launch the Send Files to TV app on your Chromecast
2. Approve any file permissions that the app requests
3. Select the `Receive` option
4. Note the IP address displayed at the bottom of the screen
5. Leave the app running, we will need it later to transfer the WireGuard configuration file

### 4. Update the WireGuard configuration with the Chromecast IP address

1. Open the `wg-client.conf` file you created in step 2.4
2. Replace `<IPAddress>` with the IP address of your Chromecast from step 3.4, the address must end with '/32' (e.g. 192.168.1.76/32)

### Transfer the WireGuard configuration to your Chromecast

1. Firstly transfer the `wg-client.conf` file to your Android phone. This can be done by uploading the file to a cloud storage service like Google Drive, and then downloading it to your phone or by using a USB cable.
2. Your TV should still have the Send Files to TV app running. If not, launch the app and select the `Receive` option.
3. Open the Send Files to TV app on your Android phone and select the `Send` option.
4. Select the `wg-client.conf` file and then select your Chromecast from the list of available devices.
5. You can now close the Send Files to TV app on your phone and TV.

### 5. Configure WireGuard on your Chromecast

1. Launch the Wireguard app on your Chromecast
2. Select the '+' icon in the bottom right to add a new tunnel
3. Select the FX File Explorer app from the list of available file managers
4. Navigate to the `Download` folder and select the `wg-client.conf` file
5. Select the `wg-client` tile to start the tunnel
6. You can now close the app, you should now see a key icon in the status bar indicating that the VPN is connected