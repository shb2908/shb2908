
```
$$\   $$\ $$\                                     
$$ |  $$ |\__|      \__$$  __|$$ |                                    
$$ |  $$ |$$\          $$ |   $$$$$$$\   $$$$$$\   $$$$$$\   $$$$$$\  
$$$$$$$$ |$$ |         $$ |   $$  __$$\ $$  __$$\ $$  __$$\ $$  __$$\ 
$$  __$$ |$$ |         $$ |   $$ |  $$ |$$$$$$$$ |$$ |  \__|$$$$$$$$ |
$$ |  $$ |$$ |         $$ |   $$ |  $$ |$$   ____|$$ |      $$   ____|
$$ |  $$ |$$ |         $$ |   $$ |  $$ |\$$$$$$$\ $$ |      \$$$$$$$\ 
\__|  \__|\__|         \__|   \__|  \__| \_______|\__|       \_______| 
```
# Technologies & Tools
[![My Skills](https://skillicons.dev/icons?i=js,docker,nodejs,python,cpp,flask,git,pytorch,ai,latex,ts,vscode,tensorflow,kubernetes,mongodb&theme=light)](https://skillicons.dev)

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Spherical Monster Eating Stars</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <style>
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      background: radial-gradient(circle at top, #020824 0%, #050510 50%, #000000 100%);
    }
    #monsterCanvas {
      width: 100vw;
      height: 100vh;
      display: block;
    }
  </style>
</head>
<body>
  <canvas id="monsterCanvas"></canvas>
  <script>
    // Polyfill for requestAnimationFrame
    (function () {
      const vendors = ['webkit', 'moz', 'ms', 'o'];
      for (let i = 0; i < vendors.length && !window.requestAnimationFrame; ++i) {
        const vp = vendors[i];
        window.requestAnimationFrame = window[vp + 'RequestAnimationFrame'];
        window.cancelAnimationFrame = window[vp + 'CancelAnimationFrame'] || window[vp + 'CancelRequestAnimationFrame'];
      }
      if (!window.requestAnimationFrame) {
        let lastTime = 0;
        window.requestAnimationFrame = function (callback) {
          const currTime = Date.now();
          const timeToCall = Math.max(0, 16 - (currTime - lastTime));
          const id = window.setTimeout(() => callback(currTime + timeToCall), timeToCall);
          lastTime = currTime + timeToCall;
          return id;
        };
        window.cancelAnimationFrame = id => clearTimeout(id);
      }
    })();

    const canvas = document.getElementById("monsterCanvas");
    const ctx = canvas.getContext("2d");
    let dpr = window.devicePixelRatio || 1;

    function resize() {
      dpr = window.devicePixelRatio || 1;
      canvas.width = innerWidth * dpr;
      canvas.height = innerHeight * dpr;
      ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    }
    window.addEventListener("resize", resize);
    resize(); // Must be called BEFORE monster/stars use innerWidth/innerHeight

    // Now safe to define monster
    const monster = {
      x: () => innerWidth / 2,
      y: () => innerHeight / 2,
      baseRadius: () => Math.min(innerWidth, innerHeight) * 0.13
    };

    const stars = [];
    const maxStars = 80;

    function rand(a, b) {
      return Math.random() * (b - a) + a;
    }

    function spawnStar() {
      const margin = 60;
      const edge = Math.floor(Math.random() * 4);
      let x, y;
      const w = innerWidth, h = innerHeight;
      if (edge === 0) { x = rand(0, w); y = -margin; }
      else if (edge === 1) { x = w + margin; y = rand(0, h); }
      else if (edge === 2) { x = rand(0, w); y = h + margin; }
      else { x = -margin; y = rand(0, h); }

      stars.push({
        x, y,
        radius: rand(3, 6),
        speed: rand(0.4, 1.0),
        twinkleSpeed: rand(2, 5),
        twinklePhase: rand(0, Math.PI * 2),
        hue: rand(45, 70),
        saturation: rand(80, 100),
        lightness: rand(60, 85),
        eaten: false
      });
    }

    function updateStars(dt) {
      dt = Math.min(dt, 100); // Cap delta to prevent spikes
      const mx = monster.x();
      const my = monster.y();
      const eatRadius = monster.baseRadius() * 0.9;

      let anyEaten = false;

      for (const s of stars) {
        const dx = mx - s.x;
        const dy = my - s.y;
        const d = Math.hypot(dx, dy); // More accurate than sqrt(dx*dx + dy*dy)
        if (d > 0.001) {
          s.x += (dx / d) * s.speed * dt * 0.07;
          s.y += (dy / d) * s.speed * dt * 0.07;
        }
        s.twinklePhase += s.twinkleSpeed * dt * 0.004;
        s.currentRadius = s.radius * (0.7 + 0.3 * Math.sin(s.twinklePhase));
        if (d < eatRadius) {
          s.eaten = true;
          anyEaten = true;
        }
      }

      // Remove eaten stars
      for (let i = stars.length - 1; i >= 0; i--) {
        if (stars[i].eaten) stars.splice(i, 1);
      }

      // Respawn
      while (stars.length < maxStars) spawnStar();

      // Update flash only if eaten this frame
      if (anyEaten) flashEnergy = 1;
    }

    function drawStar(s) {
      ctx.save();
      ctx.translate(s.x, s.y);
      const spikes = 5;
      const outer = s.currentRadius;
      const inner = outer * 0.45;
      let rot = Math.PI * 1.5;
      const step = Math.PI / spikes;
      ctx.beginPath();
      ctx.moveTo(0, -outer);
      for (let i = 0; i < spikes; i++) {
        ctx.lineTo(Math.cos(rot) * outer, Math.sin(rot) * outer);
        rot += step;
        ctx.lineTo(Math.cos(rot) * inner, Math.sin(rot) * inner);
        rot += step;
      }
      ctx.closePath();
      ctx.fillStyle = `hsl(${s.hue}, ${s.saturation}%, ${s.lightness}%)`;
      ctx.shadowColor = "white";
      ctx.shadowBlur = 10;
      ctx.fill();
      ctx.restore();
    }

    let flashEnergy = 0;

    function drawMonster(time) {
      const mx = monster.x();
      const my = monster.y();
      const base = monster.baseRadius();
      const dynamicR = base * (1 + 0.08 * Math.sin(time * 0.0024));
      const hue = 180 + 30 * Math.sin(time * 0.0016);
      const fillLightness = 55 + flashEnergy * 35;
      const color = `hsl(${hue}, 90%, ${fillLightness}%)`;
      const mouthOpen = 0.22 + 0.17 * Math.sin(time * 0.003);
      const mouthAngle = Math.PI * mouthOpen;

      ctx.save();
      ctx.translate(mx, my);

      // Outer halo
      const halo = ctx.createRadialGradient(0, 0, dynamicR * 0.4, 0, 0, dynamicR * (1.7 + flashEnergy * 0.8));
      halo.addColorStop(0, `hsla(${hue}, 100%, 70%, ${0.7 + flashEnergy * 0.3})`);
      halo.addColorStop(0.7, `hsla(${hue}, 50%, 40%, ${0.25 + flashEnergy * 0.2})`);
      halo.addColorStop(1, "rgba(0, 0, 0, 0)");
      ctx.fillStyle = halo;
      ctx.beginPath();
      ctx.arc(0, 0, dynamicR * (1.7 + flashEnergy * 0.7), 0, Math.PI * 2);
      ctx.fill();

      // Main body with mouth
      ctx.beginPath();
      ctx.arc(0, 0, dynamicR, -mouthAngle / 2, Math.PI * 2 + mouthAngle / 2);
      ctx.fillStyle = color;
      ctx.shadowBlur = 20 + flashEnergy * 70;
      ctx.shadowColor = color;
      ctx.fill();

      // Surface glow
      const glowX = Math.cos(time * 0.0015) * dynamicR * 0.4;
      const glowY = Math.sin(time * 0.0015) * dynamicR * 0.25;
      const surface = ctx.createRadialGradient(glowX, glowY, 0, glowX, glowY, dynamicR);
      surface.addColorStop(0, `rgba(255,255,255,${0.35 + flashEnergy * 0.4})`);
      surface.addColorStop(1, "transparent");
      ctx.fillStyle = surface;
      ctx.beginPath();
      ctx.arc(0, 0, dynamicR, 0, Math.PI * 2);
      ctx.fill();

      ctx.restore();
    }

    let last = performance.now();

    function loop(t) {
      const dt = Math.min(t - last, 100);
      last = t;

      // Clear with fading trail
      ctx.fillStyle = "rgba(0, 0, 0, 0.3)";
      ctx.fillRect(0, 0, innerWidth, innerHeight);

      // Reset flash energy each frame
      flashEnergy *= 0.93;
      if (flashEnergy < 0.01) flashEnergy = 0;

      updateStars(dt);
      for (const s of stars) drawStar(s);
      drawMonster(t);

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  </script>
</body>
</html>