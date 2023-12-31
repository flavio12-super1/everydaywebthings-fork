<!-- DigitalWhiteboard.svelte -->
<script>
  import { onMount } from 'svelte';

  let canvas;
  let ctx;
  let drawing = false;
  let textMode = false;
  let textX, textY;
  let backgroundColor = '#ffffff'; // Default background color
  let textColor = '#000000'; // Default text color
  let textItems = []; // Store text items
  let drawingItems = [];

  onMount(() => {
    canvas = document.getElementById('whiteboardCanvas');
    ctx = canvas.getContext('2d');
    updateCanvas();
  });

  function startDrawing(event) {
    if (textMode) return;
    drawing = true;
    const x = event.clientX - canvas.getBoundingClientRect().left;
    const y = event.clientY - canvas.getBoundingClientRect().top;
    ctx.beginPath();
    ctx.moveTo(x, y);
    drawingItems.push({ type: 'start', x, y });
  }

  function draw(event) {
    if (!drawing || textMode) return;
    const x = event.clientX - canvas.getBoundingClientRect().left;
    const y = event.clientY - canvas.getBoundingClientRect().top;
    ctx.lineTo(x, y);
    ctx.stroke();
    drawingItems.push({ type: 'line', x, y });
  }

  function endDrawing() {
    if (textMode) return;
    drawing = false;
    drawingItems.push({ type: 'end' });
  }

  function toggleTextMode() {
    textMode = !textMode;
  }

  function addText(event) {
    if (!textMode) return;
    textX = event.clientX - canvas.getBoundingClientRect().left;
    textY = event.clientY - canvas.getBoundingClientRect().top;
    const text = prompt('Enter text:');
    if (text) {
      ctx.font = '18px Arial';
      ctx.fillStyle = textColor;
      ctx.fillText(text, textX, textY);
      ctx.fillStyle = backgroundColor;
      textItems.push({ type: 'text', text, x: textX, y: textY, color: textColor });
    }
  }

  function clearWhiteboard() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    textItems = [];
    drawingItems = [];
    updateCanvas();
  }

  function updateCanvas() {
    ctx.fillStyle = backgroundColor;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = textColor;

    // Redraw text items
    for (const item of textItems) {
      ctx.font = '18px Arial';
      ctx.fillStyle = item.color;
      ctx.fillText(item.text, item.x, item.y);
      ctx.fillStyle = textColor;
    }

    // Redraw drawing items
    for (const item of drawingItems) {
      if (item.type === 'start') {
        ctx.beginPath();
        ctx.moveTo(item.x, item.y);
      } else if (item.type === 'line') {
        ctx.lineTo(item.x, item.y);
        ctx.stroke();
      } else if (item.type === 'end') {
        ctx.closePath();
      }
    }
  }

  function saveAsImage() {
    const image = canvas.toDataURL('image/png');

    const a = document.createElement('a');
    a.href = image;
    a.download = 'whiteboard.png';
    a.click();
  }
</script>

<style>
  /* Add your styles here */
  label {
    display: block;
    margin-top: 10px;
  }

  canvas {
    border: 1px solid #000;
    margin-top: 10px;
    cursor: crosshair;
  }

  button {
    margin-top: 10px;
  }

  /* Styles for the buttons */
  button {
    padding: 10px 20px;
    margin-right: 10px;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  /* Styles for the canvas */
  #whiteboardCanvas {
    border: 1px solid #000;
    position: relative;
    margin-top: 10px;
  }
</style>

<div style="position: relative;">
  <label for="bgColorPicker">Background Color:</label>
  <input type="color" id="bgColorPicker" bind:value="{backgroundColor}" on:change="{updateCanvas}" />

  <label for="textColorPicker">Text Color:</label>
  <input type="color" id="textColorPicker" bind:value="{textColor}" on:change="{updateCanvas}" />
</div>

<div style="position: relative;">
  <canvas
    id="whiteboardCanvas"
    width="800"
    height="600"
    on:mousedown="{startDrawing}"
    on:mousemove="{draw}"
    on:mouseup="{endDrawing}"
    on:click="{addText}"
  ></canvas>
</div>

<div>
  <button on:click="{toggleTextMode}">Text Mode</button>
  <button on:click="{clearWhiteboard}">Clear Whiteboard</button>
  <button on:click="{saveAsImage}">Save as Image</button>
</div>

