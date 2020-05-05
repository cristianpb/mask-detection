<main>
  <body>
    <h1>Mask detector</h1>
    <div id="loadingMessage">🎥 Unable to access video stream (please make sure you have a webcam enabled)</div>
    <canvas id="canvasResult" hidden></canvas>
    <canvas id="canvas" hidden></canvas>
  </body>
</main>

<script>
  import { onMount } from 'svelte';
  import * as cvstfjs from '@microsoft/customvision-tfjs';

  let model = new cvstfjs.ObjectDetectionModel();


  onMount(async() => {
    var video = document.createElement("video");
    var canvasElement = document.getElementById("canvas");
    var canvas = canvasElement.getContext("2d");
    var loadingMessage = document.getElementById("loadingMessage");
    await model.loadModelAsync('__BASEURL__/model/model.json');


    // Use facingMode: environment to attemt to get the front camera on phones
    navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" } }).then(function(stream) {
      video.srcObject = stream;
      video.setAttribute("playsinline", true); // required to tell iOS safari we don't want fullscreen
      video.play();
      requestAnimationFrame(tick);
    });

    async function tick() {
      loadingMessage.innerText = "⌛ Loading video..."
      if (video.readyState === video.HAVE_ENOUGH_DATA) {
        loadingMessage.hidden = true;
        canvasElement.hidden = false;

        const frame = video
        canvasElement.height = frame.videoHeight;
        canvasElement.width = frame.videoWidth;
        canvas.drawImage(frame, 0, 0, canvasElement.width, canvasElement.height);
        var imageData = canvas.getImageData(0, 0, canvasElement.width, canvasElement.height);

        const time = new Date()
        if (time.getSeconds() % 1 == 0) {

          const result = await model.executeAsync(imageData);
          const probs = result[1]
          const boxes = result[0]
          const threshold = 0.25

          if (probs.some(x => x > threshold)) {
            var canvasElementResult = document.getElementById("canvasResult");
            var canvasResult = canvasElementResult.getContext("2d");
            canvasElementResult.hidden = false;
            canvasElementResult.height = frame.videoHeight;
            canvasElementResult.width = frame.videoWidth;
            canvasResult.drawImage(frame, 0, 0, canvasElementResult.width, canvasElementResult.height);
            boxes.forEach((item, idx) => {
              if (probs[idx] > threshold) {
                const x1 = item[0] * imageData.width
                const y1 = item[1] * imageData.height
                const x2 = item[2] * imageData.width
                const y2 = item[3] * imageData.height
                canvasResult.beginPath();
                canvasResult.lineWidth = 10;
                canvasResult.strokeStyle = 'red';
                canvasResult.strokeRect(x1, y1, (x2-x1), (y2 -y1));
              }
            })
          }

        }
      }
      requestAnimationFrame(tick)
    }

  })


</script>


<style>
  body {
    font-family: 'Ropa Sans', sans-serif;
    color: #333;
    max-width: 640px;
    margin: 0 auto;
    position: relative;
  }

  h1 {
    margin: 10px 0;
    font-size: 40px;
  }

  #loadingMessage {
    text-align: center;
    padding: 40px;
    background-color: #eee;
  }

  #canvas {
    width: 100%;
  }

  #canvasResult {
    width: 100%;
  }
</style>
