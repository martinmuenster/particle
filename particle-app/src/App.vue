<template>
  <v-app>
    <v-content>
     <div id="container">
      <v-container fluid>
        <v-slide-y-transition mode="out-in">
          <v-layout column align-center>
            <h2>Audio/Visual Generation with a Particle IoT Device </h2>
            <v-btn block color="secondary" dark @click="initialize">Start</v-btn>
            <p> Light level: {{light}} | Temperature: {{temp}} | Humidity: {{humidity}} | Motion: {{motion!==0}} </p>
          </v-layout>
        </v-slide-y-transition>
      </v-container>
     </div>
    </v-content>
  </v-app>
</template>

<script>
    export default {

        data() {
            return {
                clipped: false,
                drawer: true,
                fixed: false,
                items: [{
                    icon: 'bubble_chart',
                    title: 'Inspire'
                }],
                miniVariant: false,
                right: true,
                rightDrawer: false,
                title: 'Vuetify.js',
                token: '',
                light: 10,
                temp: 1,
                humidity: 1,
                motion: 0,
                rotation: .01

            }
        },
        methods: {
            initialize() {
                this.initializeVisuals();
                this.initializeMusic();

                var n = 300;
                var i = 0;
                this.loadParticle();
                var parent = this;
                setInterval(function() {
                    i++;
                    if (i < n) {
                        parent.loop();
                    }
                }, 200);
            },

            loop() {
                this.updateParameters();
            },

            initializeVisuals() {
                var THREE = require('THREE');
                var scene = new THREE.Scene();
                var camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

                var renderer = new THREE.WebGLRenderer();
                renderer.setSize(window.innerWidth, window.innerHeight);
                document.getElementById('container').appendChild(renderer.domElement);

                var geometry = new THREE.BoxGeometry(2, 2, 2);
                var wireframe = new THREE.WireframeGeometry(geometry);

                var cube = new THREE.LineSegments(wireframe);
                cube.material.depthTest = true;
                cube.material.opacity = 1;
                cube.material.transparent = false;
                cube.material.linewidth = 2;

                /*var material = new THREE.MeshBasicMaterial({
                    color: 0x00ff00
                });
                var cube = new THREE.Mesh(geometry, material);*/
                scene.add(cube);

                camera.position.z = 4;
                
                var interval = 0;
                var parent = this;
                var animate = function() {
                    requestAnimationFrame(animate);

                    cube.rotation.x += (76 - parent.temp) / 80;
                    cube.rotation.y += (76 - parent.temp) / 80;

                    cube.scale.x = Math.abs(Math.log(parent.light) - 5);
                    cube.scale.y = (parent.humidity / 35) * (parent.humidity / 35);
                    cube.scale.z = parent.temp / 76;
                    
                    if (parent.motion !== 0) {
                        camera.position.z = 7;
                        if (interval === 0) {
                            cube.position.x = 3;
                            interval = 1;
                        }
                        else if (interval === 1) {
                            cube.position.x = 0;
                            interval = 2;
                        }
                        else {
                            cube.position.x = -3;
                            interval = 0;
                        }
                    }
                    else {
                        cube.position.x = 0;
                        camera.position.z = 4;
                    }

                    renderer.render(scene, camera);
                };

                animate();
            },

            updateInstrument() {
                this.instrument1.oscillator.modulationFrequency = this.light * 2;
                this.instrument1.detune = this.temp;
                console.log("updating instrument");
                this.synth1.set(this.instrument1);
                this.pitchShifter1.pitch = Math.log(this.light) - this.humidity / 3;
                if (this.motion !== 0) {
                    this.instrument2.volume = -10;
                    this.synth2.set(this.instrument2);
                    //this.crusher.wet.value = 1.0;
                    //console.log("Non-Zero");
                } else {
                    this.instrument2.volume = -50;
                    this.synth2.set(this.instrument2);
                    //this.crusher.wet.value = 0.0;
                    //console.log("Zero");
                }

            },

            initializeMusic() {
                //var light = this.getParameters()
                var Tone = require('tone');
                this.instrument1 = {
                    "oscillator": {
                        "type": "pwm",
                        "modulationFrequency": this.light * 500
                    },
                    "envelope": {
                        "attack": 0.02,
                        "decay": 0.1,
                        "sustain": 0.2,
                        "release": 0.9,
                    }

                };

                this.instrument2 = {
                    "oscillator": {
                        "type": "pwm",
                        "modulationFrequency": 1
                    },
                    "envelope": {
                        "attack": 0.02,
                        "decay": 0.1,
                        "sustain": 0.2,
                        "release": 0.9,
                    },
                    "volume": -8,
                };

                this.pitchShifter1 = new Tone.PitchShift().toMaster();
                this.volume1 = new Tone.Volume(-5);
                //this.crusher = new Tone.BitCrusher();




                console.log("Creating instruments...");




                this.synth1 = new Tone.Synth(this.instrument1);
                //this.synth1.connect(this.crusher);
                //this.synth1.connect(this.volume1);
                this.synth1.connect(this.pitchShifter1);
                var synth1 = this.synth1;

                /*var pattern = new Tone.Pattern(function(time, note) {
                    synth1.triggerAttack("C4", 0.25);
                }, ["C4", "E4", "G4", "A4"]);*/

                //pattern.start(0);
                //Tone.Transport.start();


                //this.synth2 = new Tone.Synth(this.instrument2).toMaster();
                //this.synth2 = new Tone.NoiseSynth().toMaster();
                this.synth2 = new Tone.Noise({
                    "volume": -10,
                    "type": "white"
                }).toMaster();
                /*this.env2 = new Tone.AmplitudeEnvelope({
                    "attack": 0.11,
                    "decay": 0.21,
                    "sustain": 0.09,
                    "release": 1.2
                }).toMaster();*/
                //this.synth2.connect(this.env2);




                this.synth1.triggerAttack("440", "+1");
                //this.synth2.triggerAttack("120", "+0");
                this.synth2.start();

            },

            getParameters() {
                this.getLight.then(function(data) {
                    console.log('Device variable retrieved successfully:', data);
                });
            },

            loadParticle() {
                var Particle = require('particle-api-js');
                this.particle = new Particle();

                this.getToken = this.particle.login({
                    username: 'martinhmuenster@gmail.com',
                    password: '@Martin196'
                });
            },

            updateParameters() {
                var token;
                var particle = this.particle;
                var parent = this;
                this.getToken.then(
                    function(data) {
                        token = data.body.access_token;
                        particle.getVariable({
                            deviceId: '3b0028000351353432383931',
                            name: 'light',
                            auth: token
                        }).then(function(data) {
                            //console.log('Device variable retrieved successfully:', data);
                            parent.lightObject = data;
                            parent.light = parent.lightObject.body.result;
                            //console.log(parent.light);
                        }, function(err) {
                            console.log('An error occurred while getting attrs:', err);
                        });
                        particle.getVariable({
                            deviceId: '3b0028000351353432383931',
                            name: 'motion',
                            auth: token
                        }).then(function(data) {
                            //console.log('Device variable retrieved successfully:', data);
                            parent.motionObject = data;
                            parent.motion = Math.log(parent.motionObject.body.result);
                            //console.log(parent.motion);
                        }, function(err) {
                            console.log('An error occurred while getting attrs:', err);
                        });
                        particle.getVariable({
                            deviceId: '3b0028000351353432383931',
                            name: 'humidity',
                            auth: token
                        }).then(function(data) {
                            //console.log('Device variable retrieved successfully:', data);
                            parent.humidityObject = data;
                            parent.humidity = parent.humidityObject.body.result;
                            //console.log(parent.humidity);
                        }, function(err) {
                            console.log('An error occurred while getting attrs:', err);
                        });
                        particle.getVariable({
                            deviceId: '3b0028000351353432383931',
                            name: 'temp',
                            auth: token
                        }).then(function(data) {
                            //console.log('Device variable retrieved successfully:', data);
                            parent.tempObject = data;
                            parent.temp = parent.tempObject.body.result;
                            //console.log(parent.temp);
                            parent.updateInstrument();
                        }, function(err) {
                            console.log('An error occurred while getting attrs:', err);
                        });

                    },
                    function(err) {
                        console.log('Could not log in.', err);
                    }
                );

            },

            stop() {
                this.synth1.stop;
            }

        }
    }

</script>

<style>
    #container {
        margin: 0;
    }

    canvas {
        width: 100%;
        height: 100%;
    }
    p {
        font-weight: 200
    }

</style>
