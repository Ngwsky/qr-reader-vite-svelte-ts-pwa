<script lang="ts">
  import { onMount } from 'svelte'
  import jsQR from "jsqr"

  let canvasElement
  let videoElement
  let animationid = null
  let qrValues = []

  function tick() {
    if (videoElement.readyState === videoElement.HAVE_ENOUGH_DATA) {
      canvasElement.height = videoElement.videoHeight
      canvasElement.width = videoElement.videoWidth

      const ctx = canvasElement.getContext("2d")
      ctx.drawImage(videoElement, 0, 0, canvasElement.width, canvasElement.height)
      const imageData = ctx.getImageData(0, 0, canvasElement.width, canvasElement.height)
      const code = jsQR(imageData.data, imageData.width, imageData.height, {
        inversionAttempts: "dontInvert",
      })

      if (code) {
        drawline(ctx, code.location.topLeftCorner, code.location.topRightCorner, "#FF3B58")
        drawline(ctx, code.location.topRightCorner, code.location.bottomRightCorner, "#FF3B58")
        drawline(ctx, code.location.bottomRightCorner, code.location.bottomLeftCorner, "#FF3B58")
        drawline(ctx, code.location.bottomLeftCorner, code.location.topLeftCorner, "#FF3B58")
        if (code.data && (qrValues.length < 1 || qrValues[0] != code.data)) {
          qrValues = [code.data, ...qrValues]
          console.log(code)
        }
        animationid = requestAnimationFrame(tick)
        // cancelAnimationFrame(animationid)
      } else {
        animationid = requestAnimationFrame(tick)
      }
    } else {
      animationid = requestAnimationFrame(tick)
    }
  }

  function drawline(ctx, begin, end, color) {
    ctx.beginPath()
    ctx.moveTo(begin.x, begin.y)
    ctx.lineTo(end.x, end.y)
    ctx.lineWidth = 4
    ctx.strokeStyle = color
    ctx.stroke()
  }

  onMount(() => {
    navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" } }).then((stream) => {
      videoElement.srcObject = stream
      videoElement.setAttribute("playsinline", true)
      videoElement.play()
      animationid = requestAnimationFrame(tick)
    })
	})
</script>

<div class="row">
  <div class="column">
    <canvas class="w100" style="max-width:640px;" bind:this={canvasElement}></canvas>
  </div>
</div>
<video bind:this={videoElement} hidden></video>
<div class="row">
  <div class="column">
    <ul>
      {#each qrValues as qrValue}<li>{qrValue}</li>{/each}
    </ul>
  </div>
</div>

<style scoped>
.w100 {
  width: 100%;
}
</style>
