<script context="module">
  import { onMount } from "svelte";
  const [width, height] = [640, 200];
  const backgroundColor = "#222831";
  let canvas;

  const dataLimit = 100;

  const data = [];

  const clear = () => {
    const ctx = canvas.getContext("2d");
    ctx.fillStyle = backgroundColor;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  };

  export const addDataPoint = point => {
    data.push(point);
    if (data.length > dataLimit) data.shift();
    drawPlot();
  };

  const drawPlot = () => {
    clear();
    const ctx = canvas.getContext("2d");

    ctx.beginPath();
    ctx.strokeStyle = "orange";
    const w = width * (2 / 3);
    ctx.moveTo(0, height - 3 * data[0].recFreq);
    for (let i = 1; i < data.length; i++) {
      const x = (i * w) / dataLimit;
      const y = height - 3 * data[i].recFreq;
      ctx.lineTo(x, y);
      ctx.moveTo(x, y);
    }
    ctx.stroke();
  };
</script>

<canvas bind:this={canvas} {width} {height} />
