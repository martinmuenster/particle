# Audio/Visual Generation with a Particle IoT Device
### Final project for Generative Media Authorship
#### Martin Muenster

### Introduction
This project was built to capture changing environments--weather, social gatherings, nature--and attempts to represent these environment through computer generated audio and visuals. 

### Setup
The project was created using a SparkFun Photon Redboard. Three sensors were connected to the board: light, motion, and humidity/temperature. The code used to run the device is linked here:
[Particle code](https://github.com/martinmuenster/particle/env.ino)
An image of the setup is shown below:
![alt text](https://github.com/martinmuenster/particle/images/setup.jpg "Particle setup")

### Website Funcionality
This project uses Vuetify for the front-end. The Particle JavaScript API was used to allow the data from IoT device to be read. Tone.js was used to generate the audio, and THREE.js was used to generate the visuals. Light, humidity, temperature, and motion data are recieved from the Particle, and are used to change parameters within the visual and audio interfaces. Particle variables can easily be mapped to different audio/visual parameters, and new features/sensors can be easily implemented.

### Use
To use, make sure the Particle device has the specified hardware setup, the required .ino file, is connected to power, and is connected to the internet. On [this site](https://martinmuenster.github.io/particle/particle-app/index.html) press the large "Start" button.

### Bugs
There have been numerous issues with the reliability and speed of the connection between the Particle and the website. There are fairly large lag times between data updates, caused by the time of the get requests to the Particle. Usually these updates occur around once per second. Fairly often, the particle is disconnected, or the data from the particle is unavailable. This results in long periods of unchanged audio/visuals, or audio/visual that hasn't been initialized. This issue can be fixed by waiting for the Particle to start responding, or by reseting the particle.