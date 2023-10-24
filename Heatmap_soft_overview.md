# AAC Solutions Heat Mapping Notes

This document is centered around Jantman's Python Software and Docker Image.

## FOSS Overview

This document aims to provide technicians with detailed instructions on WiFi heatmap software. The software is an open-source (AGPL-3.0) python run WiFi heat mapping tool, designed to map out connectivity, speed, and overall strength of WiFi across a venue, building, or site with a provided floor plan.

## Requirements

1. Knowledge of Python
2. Knowledge of Linux Networking 
3. Familiarity with Raspberry Pi / Pi Systems and Linux Machines 
4. Basic understanding of WiFi / Radio Physics
5. Moderate knowledge on Docker / Operations with Docker   

## How the Software Works

1. A small Linux machine is set up on the network in a fixed physical location.
   - This machine hosts an iperf server acting as a (Home Base) for the mapping software to communicate with.
2. Python mapping software is set up on a laptop or other Linux device.
   - This machine measures RSSI, Connection speed, and Bandwidth across the network by communicating with the machine set up in step one.
3. A detailed image/photo of a floor plan is required to act as a background.

## Setup Guide

### 1. iperf Server Setup (Raspberry Pi)

1. Install iperf3 on your Raspberry Pi:

    ```bash
    sudo apt-get install iperf3
    ```

2. Start the iperf3 server on your Raspberry Pi:

    ```bash
    iperf3 -s
    ```

3. Note the IP address of the Raspberry Pi. 

    ```bash 
    ip a 
    ```

    Or if no display / display server is present you can manually search for the Raspberry Pi using the other Machine and: 

    ```bash 
    nmap -T5 -sn  <subnet / 192.169.1.0/24> && nmap -T5 -Pn -O <subnet / 192.168.1.0/24> >> nmap_output<location>.txt
    ```

    Substituting the command as you see fit. 


### 2. Survey Machine Setup (Laptop)

1. **Installation and Dependencies:**

    The recommended installation is via Docker to avoid manual dependency installations. 
    However, if you prefer manual installation, follow the steps below:

    - **iperf3:** 
        ```bash
        sudo pacman -S iperf3
        ```

    - **libiw:** (Might need to search in the AUR or find an alternative installation method)
        
    - **wxPython Phoenix:** Install from the AUR as python-wxpython.
        ```bash
        sudo pacman -S yay && yay -Syu python-wxpython
        ```

2. **Performing a Survey:**

    - Connect to the network you want to survey.
    - Run the following command to start the survey:
        ```bash
        sudo wifi-survey -i <INTERFACE> -p <FLOORPLAN PNG> -t <TITLE> -s <IPERF3_SERVER>
        ```

3. **Heatmap Generation:**

    - Once you've completed the survey, run the following command to generate the heatmap files:
        ```bash
        wifi-heatmap <TITLE>
        ```

4. **Running In Docker (Recommended):**

    - **Survey:**
        ```bash
        docker run \
          --net="host" \
          --privileged \
          --name survey \
          -it \
          --rm \
          -v $(pwd):/pwd \
          -w /pwd \
          -e DISPLAY=$DISPLAY \
          -v "$HOME/.Xauthority:/root/.Xauthority:ro" \
          jantman/python-wifi-survey-heatmap \
          wifi-survey -b <BSSID> -i <INTERFACE> -s <IPERF SERVER> -p <FLOORPLAN PNG> -t <TITLE>
        ```

    - **Heatmap:**
        ```bash
        docker run -it --rm -v $(pwd):/pwd -w /pwd jantman/python-wifi-survey-heatmap:23429a4 wifi-heatmap <TITLE>
        ```

    - **iperf3 server:**
        ```bash
        docker run -it --rm -p 5201:5201/tcp -p 5201:5201/udp jantman/python-wifi-survey-heatmap iperf3 -s
        ```

Replace `<BSSID>, <INTERFACE>, <IPERF SERVER>, <FLOORPLAN PNG>, and <TITLE>` with your network's BSSID, your wireless interface name, the IP address of your iperf3 server, the path to your floorplan image, and a title for your survey, respectively.

These instructions should aid in setting up the necessary environment to perform a WiFi survey and generate heatmaps of a room or building.


## Issues

### Docker Accessibility Bus Connection Issue

If you encounter the following error:

```plaintext
Couldn't connect to accessibility bus: Failed to connect to socket /run/user/1000/at-spi/bus_0: No such file or directory

when running in Docker, it's necessary to mount the socket in Docker explicitly by adding an additional '-v' switch:
```

```bash

docker run ... -v /run/user/1000/at-spi/bus_0:/run/user/1000/at-spi/bus_0 ...

```


## Legal Acknowledgements

This project utilizes the software provided by Jantman's python-wifi-survey-heatmap, which is licensed under the GNU Affero General Public License v3.0 (AGPL-3.0). Adherence to the terms and conditions of the AGPL-3.0 license is required when using, modifying, or distributing the software or any derivative works based on it.

Additionally, any publication or distribution of the work or derivatives from the work should provide proper accreditation to the original source and its contributors.

For the complete terms and licensing information, refer to the [AGPL-3.0 License](https://www.gnu.org/licenses/agpl-3.0.en.html) and the [project's repository](https://github.com/jantman/python-wifi-survey-heatmap).






