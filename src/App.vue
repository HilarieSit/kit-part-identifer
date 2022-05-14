<template>
<div id="app">
  <h1> Kit Part Identification </h1>
  <div id="menu">
    <select id="camera_selected" v-if="cameraOn == false">
      <option v-for="device in devices" :key="device.deviceId" :value="device.deviceId">{{device.label}}</option>
    </select>
    <button class="btn btn-success" v-if="cameraOn == false" @click="startStreaming">Start Camera</button>
    <button class="btn btn-danger" v-if="cameraOn == true" @click="stopStreaming">Stop Camera</button>
    <button class="btn btn-success" v-if="cameraOn == true" @click="detectObjects">Predict</button>
  </div>
  <div id="video">
    <div id="overlay"><p v-if="prediction != 'None'">Prediction: {{prediction}}, Accuracy: {{accuracy}}</p></div>
    <video
        class="w-full h-full mx-auto"
        autoplay
        playsinline
      >
      </video>
  </div>
</div>
</template>

<script>
var classesdict = {
  0: "Softpot",
  1: "Force Sensor",
  2: "Bend Sensor"
};

export default {
  data() {
    return { 
      devices: {},
      devices: {},
      camera: null,
      video: null,
      prediction: 'None',
      accuracy: 'None',
      cameraOn: false
    }
  },
  async mounted(){
    if ("mediaDevices" in navigator && "getUserMedia" in navigator.mediaDevices) {
        const devices_list = await navigator.mediaDevices.enumerateDevices();
        for (let i = 0; i < devices_list.length; i++) {
          if (devices_list[i].kind == "videoinput"){
            this.devices[i] = devices_list[i]
          }
        }
    }
  },
  methods:{
    startStreaming(){
      this.camera = document.getElementById("camera_selected").value;
      navigator.mediaDevices.getUserMedia({
        video: {
          deviceId: this.camera,
        },
      }).then(stream => {
        const videotag = document.querySelector("video");
        videotag.srcObject = stream;
        this.video = videotag;
        this.cameraOn = true
      });
    },
    async detectObjects(){
      const model = await tf.loadLayersModel('src/model/model.json');
      const res = tf.tidy(() => {
        var example = tf.browser.fromPixels(this.video);  // for example
        example = tf.image.resizeBilinear(example, [224, 224])
        const pred = model.predict(example.expandDims(0))
        const pred_arr = pred.arraySync()[0]
        tf.dispose(example)
        tf.dispose(pred)
        let i = pred_arr.indexOf(Math.max(...pred_arr));
        this.prediction = classesdict[i]
        this.accuracy = pred_arr[i].toPrecision(3)
      })
    },
    stopStreaming(){
      this.video.srcObject.getTracks().forEach(function(track) {
          track.stop();
      });
      this.cameraOn = false
    }
  }
}
</script>

<style>
h1{
  padding-top: 25px;
}
#app{
  text-align: center;
  height: 100%;
}
video{
  max-width: 400px;
  width: 100%;
  height: 400px;
  background-color: black;
  padding: 0px;
}
#video{
  position: fixed;
  width: 100%;
  margin: 0px auto;
  text-align: center;
}
p{
  color: white;
}
.btn{
  margin: 0 2.5px;
}
#menu{
  vertical-align: text-top;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 10px;
}
#overlay{
  position: absolute;
  top: 20px;
  left: 0;
  width: 100%;
  text-align: center;
}
select{
  height: 38px;
  border: 2px solid #aaa;
  border-radius: 5px;
  margin: 0 2.5px !important;
}
</style>


