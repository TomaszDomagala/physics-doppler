<script>
  import { onMount } from "svelte";
  import Plot, { addDataPoint } from "./Plot.svelte";

  const [width, height] = [640, 480];
  const plotHeight = [240];

  let canvas;
  const backgroundColor = "#30475e";

  const maxRadius = 600;
  const maxTime = 5000; //5 sec

  let mousePos = { x: 0, y: 0 };
  let mouseDown = false;

  const receiver = { x: width / 2, y: height / 2 };

  const frequency = 15;
  const signalSpeed = 400;

  let signalTimeoutId;

  let mouseInside = false;
  let sources = [];

  const drawReceiver = () => {
    const ctx = canvas.getContext("2d");
    ctx.beginPath();
    ctx.arc(receiver.x, receiver.y, 7, 0, 2 * Math.PI);
    ctx.fillStyle = "orange";
    ctx.fill();
  };

  const drawCircle = source => {
    const { x, y, t } = source;

    const deltaT = (Date.now() - t) / 1000;
    const r = signalSpeed * deltaT;

    const ctx = canvas.getContext("2d");
    const alpha = 1 - (deltaT * 1000) / maxTime;
    const circleColor = `rgba(236,236,236,${alpha})`;
    ctx.beginPath();

    ctx.strokeStyle = circleColor;
    ctx.arc(x, y, r, 0, 2 * Math.PI);
    ctx.stroke();
  };

  const clear = () => {
    const ctx = canvas.getContext("2d");
    ctx.fillStyle = backgroundColor;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  };

  const onMouseEnter = () => {
    mouseInside = true;
    startSendingSignal();
  };
  const onMouseLeave = () => {
    mouseInside = false;
  };

  const onMouseDown = () => {
    mouseDown = true;
    startSendingSignal();
  };

  const onMouseUp = () => {
    mouseDown = false;
  };

  const startSendingSignal = () => {
    if (!(mouseInside && mouseDown)) {
      clearTimeout(signalTimeoutId);
      return;
    }

    sources.push({ x: mousePos.x, y: mousePos.y, t: Date.now() });

    signalTimeoutId = setTimeout(startSendingSignal, 1000 / frequency);
  };

  onMount(() => {
    document.body.onmousedown = onMouseDown;
    document.body.onmouseup = onMouseUp;

    clear();
    window.requestAnimationFrame(frame);
  });

  let time = Date.now();
  let velocity = 0;
  let angle = 0;
  let prevMousePos = { x: 0, y: 0 };

  const calculateVelocity = () => {
    const now = Date.now();
    const deltaTime = (now - time) / 1000;

    const dx = mousePos.x - prevMousePos.x;
    const dy = mousePos.y - prevMousePos.y;

    const deltaDistance = Math.sqrt(Math.pow(dx, 2) + Math.pow(dy, 2));
    velocity = deltaDistance / deltaTime;
    angle = Math.atan2(dy, dx);
    time = now;
    prevMousePos = { ...mousePos };
  };

  let receivedFrequency = 0;

  const sendingSignal = () => mouseDown && mouseInside;
  const handleMouseMove = event => {
    const rect = event.target.getBoundingClientRect();

    mousePos.x = event.clientX - rect.left;
    mousePos.y = event.clientY - rect.top;
  };

  const calculateReceivedFrequency = () => {
    const receiverAngle = Math.atan2(
      receiver.y - mousePos.y,
      receiver.x - mousePos.x
    );

    receivedFrequency =
      (frequency * signalSpeed) /
      (signalSpeed - velocity * Math.cos(receiverAngle - angle));

    addDataPoint({ recFreq: mouseDown && mouseInside ? receivedFrequency : 0 });
  };

  const frame = () => {
    calculateVelocity();
    calculateReceivedFrequency();
    const now = Date.now();

    sources = sources.filter(({ t }) => now - t < maxTime);

    clear();

    // if (mouseInside && mouseDown) {
    //   sources.push({ x: mousePos.x, y: mousePos.y, r: 0 });
    // }
    drawReceiver();
    sources.forEach(drawCircle);

    window.requestAnimationFrame(frame);
  };
</script>

<style>
  p {
    color: #ececec;
    margin-left: 20px;
  }
</style>

<div>
  <div style="text-align:center;">
    <canvas
      bind:this={canvas}
      on:mouseenter={onMouseEnter}
      on:mouseleave={onMouseLeave}
      on:mousemove={handleMouseMove}
      {width}
      {height} />

  </div>
  <div style="text-align:center;">
    <Plot />
  </div>
  <!-- <p>x: {mousePos.x} y: {mousePos.y}</p> -->
  <p>częstotliwość źródła: {mouseDown && mouseInside ? frequency : 0} Hz</p>
  <p>
    częstotliwość odebrana: {mouseDown && mouseInside ? Math.round(receivedFrequency) : 0}
    Hz
  </p>
  <br />
  <p>
    Proszę najechać kursorem na planszę i przytrzymać lewy przycisk myszki, aby
    zacząć wysyłać fale.
  </p>
  <p>
    Zjawisko Dopplera polega na powstawaniu różnicy w częstotliwości fali
    wysyłanej przez źródło oraz częstotliwości rejestrowanej przez odbiornik.
    Różnica powstaje, ponieważ prędkość fali nie zależy od prędkości jej źródła.
    Powyższa symulacja przedstawia grzbiety wysyłanej fali jako białe okręgi.
    Gdy poruszymy myszką z odpowiednią prędkością, możemy zwiększyć zagęszczenie
    grzbietów fali w danym kierunku, czyli zwiększyć częstotliwość w tym
    miejscu.
  </p>
  <p>
    Pomarańczowa kropka na środku planszy to odbiornik. Gdy wysyłamy falę i
    poruszamy się w jego kierunku, odbiera falę o większej częstotliwości. Gdy
    oddalamy się od niego, odbiera falę o częstotliwości mniejszej.
  </p>

</div>
