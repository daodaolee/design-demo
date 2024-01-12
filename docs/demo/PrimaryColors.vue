<template>
  <div class="primaryColor">
    <canvas ref="canvas" width="200" height="200"></canvas>
    <div class="panel">
      <div class="oper">
        <div class="oper-item" v-for="item in  Object.keys(rangeDom) " :key="item">
          <label :for="item">{{ item }}</label>
          <span>00</span>
          <input :name="item" type="range" min="0" max="254" step="1" v-model="rangeDom[item]['percent']" list="values" />
          <span>FF</span>
          <div class="preview" :style="{ background: getColor(item, hexParts[rangeDom[item].percent]) }"></div>
          <span>（{{ getColor(item, hexParts[rangeDom[item].percent]) }}）</span>
        </div>
      </div>
      <div class="compose">
        <div class="compose-item">
          <div :style="{ background: getColor('red', hexParts[rangeDom['red'].percent]) }"></div> +
          <div :style="{ background: getColor('lime', hexParts[rangeDom['lime'].percent]) }"></div> =
          <div :style="{ background: `#${hexParts[rangeDom['red'].percent]}${hexParts[rangeDom['lime'].percent]}00` }">
          </div>
          <span>#{{ hexParts[rangeDom['red'].percent] }}{{ hexParts[rangeDom['lime'].percent] }}00</span>
        </div>

        <div class="compose-item">
          <div :style="{ background: getColor('red', hexParts[rangeDom['red'].percent]) }"></div> +
          <div :style="{ background: getColor('blue', hexParts[rangeDom['blue'].percent]) }"></div> =
          <div :style="{ background: `#${hexParts[rangeDom['red'].percent]}00${hexParts[rangeDom['blue'].percent]}` }">
          </div>
          <span>#{{ hexParts[rangeDom['red'].percent] }}00{{ hexParts[rangeDom['blue'].percent] }}</span>
        </div>

        <div class="compose-item">
          <div :style="{ background: getColor('lime', hexParts[rangeDom['lime'].percent]) }"></div> +
          <div :style="{ background: getColor('blue', hexParts[rangeDom['blue'].percent]) }"></div> =
          <div :style="{ background: `#00${hexParts[rangeDom['lime'].percent]}${hexParts[rangeDom['blue'].percent]}` }">
          </div>
          <span>#00{{ hexParts[rangeDom['lime'].percent] }}{{ hexParts[rangeDom['blue'].percent] }}</span>
        </div>
        <div class="compose-item">
          <div :style="{ background: getColor('red', hexParts[rangeDom['red'].percent]) }"></div> +
          <div :style="{ background: getColor('lime', hexParts[rangeDom['lime'].percent]) }"></div> +
          <div :style="{ background: getColor('blue', hexParts[rangeDom['blue'].percent]) }"></div> =
          <div
            :style="{
              background: `#${hexParts[rangeDom['red'].percent]}${hexParts[rangeDom['red'].percent]}${hexParts[rangeDom['lime'].percent]}`
            }">
          </div>
          <span>#{{ hexParts[rangeDom['red'].percent] }}{{ hexParts[rangeDom['lime'].percent] }}{{
            hexParts[rangeDom['blue'].percent] }}</span>
        </div>
      </div>
    </div>


    <datalist id="values">
      <option :value="item" :label="item" v-for=" item  in  calibration " :key="item"></option>
    </datalist>
  </div>
</template>
<script  setup>
import { ref, onMounted, watch, reactive } from "vue"

let rangeDom = reactive({
  'red': {
    x: 70,
    y: 120,
    percent: 254
  },
  'lime': {
    x: 100,
    y: 90,
    percent: 254
  },
  'blue': {
    x: 130,
    y: 120,
    percent: 254
  }
})
const calibration = Array.from({ length: 6 }, (_, index) => index * 51);
const canvas = ref(null)
let ctx = null
function drawCircle(x, y, r, color) {
  ctx.beginPath()
  ctx.arc(x, y, r, 0, 2 * Math.PI)
  ctx.fillStyle = color
  ctx.fill()
}
// 00到FF的数组
function splitHexRangeInto100Parts() {
  const startHex = 0x00;
  const endHex = 0xFF;
  const totalParts = 255;
  const partSize = (endHex - startHex) / (totalParts - 1);
  const hexParts = [];

  for (let i = 0; i < totalParts; i++) {
    const currentPart = Math.round(startHex + i * partSize);
    const hexValue = currentPart.toString(16).toUpperCase().padStart(2, '0');
    hexParts.push(hexValue);
  }
  return hexParts;
}
function getColor(data, target) {
  switch (data) {
    case "red":
      return `#${target}0000`
    case "lime":
      return `#00${target}00`
    case "blue":
      return `#0000${target}`
  }
}

const hexParts = splitHexRangeInto100Parts();

function draw() {
  Object.keys(rangeDom).map(item => {
    const target = hexParts[rangeDom[item].percent]
    ctx.clearRect(0, 0, 200, 200)
    requestAnimationFrame(() => {
      drawCircle(rangeDom[item].x, rangeDom[item].y, 50, getColor(item, target))
    })
  })
}
watch(rangeDom, () => {
  draw()
})

onMounted(() => {
  ctx = canvas.value.getContext("2d")
  canvas.value.width = 200
  canvas.value.height = 200
  ctx.globalCompositeOperation = 'screen'
  draw()
})

</script>
<style scoped>
.primaryColor {
  display: flex;
  align-items: center;
}

canvas {
  background-color: #000;
  margin-right: 20px;
}

.panel {
  width: 450px;
  height: 200px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}


.oper-item {
  height: 30px;
  display: flex;
  align-items: center;
  font-size: 14px;
}

input {
  width: 200px;
  cursor: pointer;
}

label {
  display: inline-block;
  width: 40px;
}

.preview {
  width: 12px;
  height: 12px;
  margin-left: 10px;
  border-radius: 50%;
}

.compose {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
}

.compose-item {
  display: flex;
  align-items: center;
  margin-right: 10px;
  font-size: 12px;
}

.compose-item div {
  width: 12px;
  height: 12px;
  margin: 0 5px;
  border-radius: 50%;
}
</style>