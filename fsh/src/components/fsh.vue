<template>
  <div>
    <div
      ref="fsh"
      id="fsh"
      aria-hidden="true"
      @click="(puff = true), (puffStart = frameCount), (initPuff = true)"
    ></div>
  </div>
</template>

<script setup>
  import { onMounted, useTemplateRef } from "vue";

  const fsh = useTemplateRef("fsh");
  let puff = false;
  let initPuff = false;

  let reduceMotion = false;
  let persistPosition = true;
  let fshPosX = 64;
  let fshPosY = 64;
  let nextPosX = 64;
  let nextPosY = 64;
  let frameCount = 0;
  let idleTime = 0;
  let fshSpeed = 10;
  let direction = "leftUn";
  let puffStart;
  const spriteSets = {
    rightUn: [
      [0, 0],
      [-1, 0],
    ],
    leftUn: [
      [0, -1],
      [-1, -1],
    ],
    rightPuffIns: [[-2, 0]],
    leftPuffIns: [[-2, -1]],
    rightPuff: [
      [-3, 0],
      [-4, 0],
    ],
    leftPuff: [
      [-3, -1],
      [-4, -1],
    ],
  };

  onMounted(() => {
    if (
      window.matchMedia(`(prefers-reduced-motion: reduce)`) == true ||
      window.matchMedia(`(prefers-reduced-motion:reduce)`).matches === true
    ) {
      reduceMotion = true;
    }
    startFish();
  });

  function startFish() {
    if (reduceMotion) {
      return;
    } else {
      const curS = document.currentScript;
      if (curS && curS.dataset.persistPosition) {
        if (curS.dataset.persistPosition === "") {
          persistPosition = true;
        } else {
          persistPosition = JSON.parse(
            curS.dataset.persistPosition.toLowerCase()
          );
        }
      }

      if (persistPosition) {
        let fshStore = JSON.parse(window.localStorage.getItem("fsh"));
        if (fshStore !== null) {
          fshPosX = fshStore.fshPosX;
          fshPosY = fshStore.fshPosY;
          frameCount = fshStore.frameCount;
          idleTime = fshStore.idleTime;
          fsh.value.style.backgroundPosition = fsh.bgPos;
        }
      }
      fsh.value.style.left = `${fshPosX - 32}px`;
      fsh.value.style.top = `${fshPosY - 32}px`;

      if (persistPosition) {
        window.addEventListener("beforeunload", function (event) {
          this.window.localStorage.setItem(
            "fsh",
            JSON.stringify({
              fshPosX: fshPosX,
              fshPosY: fshPosY,
              frameCount: frameCount,
              idleTime: idleTime,
              bgPos: fsh.value.style.backgroundPosition,
            })
          );
        });
      }

      window.requestAnimationFrame(onAnimationFrame);
    }

    let lastFrameTimeStamp;
    function onAnimationFrame(timestamp) {
      if (fsh) {
        if (!lastFrameTimeStamp) {
          lastFrameTimeStamp = timestamp;
        }
        if (timestamp - lastFrameTimeStamp > 100) {
          lastFrameTimeStamp = timestamp;
          frame();
        }
        window.requestAnimationFrame(onAnimationFrame);
      }
    }

    function setSprite(name, frame) {
      const sprite = spriteSets[name][frame % spriteSets[name].length];
      fsh.value.style.backgroundPosition = `${sprite[0] * 64}px ${
        sprite[1] * 64
      }px`;
    }

    function idle() {
      idleTime += 1;
      puffTime();
      setSprite(direction, frameCount);
      if (idleTime > 10 && Math.floor(Math.random() * 50) === 0) {
        nextPosX = Math.floor(Math.random() * window.innerWidth);
        nextPosY = Math.floor(Math.random() * window.innerHeight);
      }
    }

    function frame() {
      frameCount += 1;
      const diffX = fshPosX - nextPosX;
      const diffY = fshPosY - nextPosY;
      const distance = Math.sqrt(diffX ** 2 + diffY ** 2);
      if (distance < fshSpeed || distance < 48) {
        idle();
        return;
      }
      if (initPuff) {
        direction = diffX / distance > 0.1 ? "leftPuffIns" : "rightPuffIns";
        direction = diffX / distance < -0.1 ? "rightPuffIns" : "leftPuffIns";
        initPuff = false;
      } else if (puff) {
        direction = diffX / distance > 0.1 ? "leftPuff" : "rightPuff";
        direction = diffX / distance < -0.1 ? "rightPuff" : "leftPuff";
      } else {
        direction = diffX / distance > 0.1 ? "leftUn" : "rightUn";
        direction = diffX / distance < -0.1 ? "rightUn" : "leftUn";
      }

      setSprite(direction, frameCount);
      puffTime();
      fshPosX -= (diffX / distance) * fshSpeed;
      fshPosY -= (diffY / distance) * fshSpeed;
      fshPosX = Math.min(Math.max(32, fshPosX), window.innerWidth - 32);
      fshPosY = Math.min(Math.max(32, fshPosY), window.innerHeight - 32);
      fsh.value.style.left = `${fshPosX - 32}px`;
      fsh.value.style.top = `${fshPosY - 32}px`;
    }

    function puffTime() {
      if (puff) {
        if (frameCount - puffStart > 20) {
          puff = false;
        }
      }
    }
  }
</script>

<style scoped>
  #fsh {
    width: 64px;
    height: 64px;
    position: fixed;
    image-rendering: pixelated;
    z-index: 33550336;
    background-image: url("../assets/fsh2.gif");
  }
</style>
