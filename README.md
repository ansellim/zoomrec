<h1 align="center">
    zoomrec	
</h1>

<h4 align="center">
	A all-in-one solution to automatically join and record Zoom meetings.
</h4>

<h3>
	Comment by Ansel: This is a fork of the zoomrec repository. I am allowing meetings to be specified based on their actual timings/dates, instead of recurrent meetings that occur "every Wednesday" or something like that.
</h3>

Join with ID and Passcode           |  Join with URL
:-------------------------:|:-------------------------:
![](doc/demo/join-meeting-id.gif)  |  ![](doc/demo/join-meeting-url.gif)

## Installation

### Install the Docker image
```bash
# Build new image using customized Dockerfile
docker build --tag zoomrec .
```

### Create folders and set permissions (on Host)

To have access to the recordings, a volume is mounted, so you need to create a folder that container users can access. Do the following on your host machine (that is, outside the Docker container).

```bash
mkdir -p example/audio

mkdir -p recordings/screenshots
chown -R 1000:1000 recordings

mkdir -p audio
chown -R 1000:1000 audio
```

Copy the contents of the `example` folder inside this repository INTO the `example` folder created above.

Edit the `meetings.csv` file according to your actual meeting schedule.

### Run the container on Linux / macOS

```bash
docker run -d --restart unless-stopped \
  -e TZ=Asia/Singapore \
  -e DISPLAY_NAME="AnselLim" \
  -v $(pwd)/recordings:/home/zoomrec/recordings \
  -v $(pwd)/example/audio:/home/zoomrec/audio \
  -v $(pwd)/example/meetings.csv:/home/zoomrec/meetings.csv:ro \
  -p 5901:5901 \
  --security-opt seccomp:unconfined \
zoomrec
```

## Preparation






## Usage

- Container saves recordings at **/home/zoomrec/recordings**
- The current directory is used to mount **recordings**-Folder, but can be changed if needed
    - Please check use of paths on different operating systems!
    - Please check permissions for used directory!
- Container stops when Python script is terminated
- Zoomrec uses a CSV file with entries of Zoom meetings to record them
    - The csv can be passed as seen below (mount as volume or add to docker image)
- To "say" something after joining a meeting:
    - ***paplay*** (*pulseaudio-utils*) is used to play a sound to a specified microphone output, which is mapped to a
      microphone input at startup.
    - ***paplay*** is triggered and plays a random file from **/home/zoomrec/audio**
    - Nothing will be played if directory:
        - does not contain **.wav** files
        - is not mounted properly

### CSV structure

CSV must be formatted as in example/meetings.csv

- Delimiter must be a semicolon "**;**"
- Only meetings with flag "**record = true**" are joined and recorded
- "**description**" is used for filename when recording
- "**duration**" in minutes (+5 minutes to the end)

weekday | time | duration | id | password | description | record | date
-------- | -------- | -------- | -------- | -------- | -------- | --------
monday | 09:55 | 60 | 111111111111 | 741699 | Important_Meeting | true | 15/01/2024 
monday | 14:00 | 90 | 222222222222 | 321523 | Unimportant_Meeting | false | 15/01/2024
tuesday| 17:00 | 90 | https://zoom.us/j/123456789?pwd=abc || Meeting_with_URL | true | 16/01/2024

