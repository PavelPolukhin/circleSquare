<template>
  <div class="circle-square">
    <div class="canvas-wrapper">
      <canvas id="myCanvas" @mousedown="toggleMouseDragging(true)" @mousemove="dragSquare"
              @mouseup="toggleMouseDragging(false)" :width="canvasWidth" :height="canvasHeight"></canvas>
    </div>
    <div class="controls">
      <label>Радиус окружности:</label>
      <input type="number" v-model="circleSize" @input="draw"/>
      <label>Размер квадрата:</label>
      <input type="number" v-model="squareSize" @input="draw"/>
      <label>X-координата квадрата:</label>
      <input type="range" :min="minSquareXY" :max="maxSquareXY" v-model="squareX" @input="draw"/>
      <label>Y-координата квадрата:</label>
      <input type="range" :min="minSquareXY" :max="maxSquareXY" v-model="squareY" @input="draw"/>
      <label>Альтернативный режим</label>
      <input type="checkbox" v-model="alternativeMode">
    </div>
  </div>
</template>

<script>
import {computed, onMounted, ref, watch} from 'vue';
import {getMousePos} from "../helpers/getMousePos.js";

export default {
  name: "CircleSquare",
  setup() {
    const canvas = ref(null);
    const ctx = ref(null);
    const alternativeMode = ref(false);
    const canvasHeight = ref(800);
    const canvasWidth = ref(800)

    // Переменные для размера окружности, размера стороны квадрата
    const circleSize = ref(400);
    const squareSize = ref(50);

    //Установка квадрата во центру круга
    const setSquareOnCenter = () => {
      squareX.value = circleSize.value;
      squareY.value = circleSize.value;
    }

    //Координаты квадрата
    const squareX = ref(null);
    const squareY = ref(null);

    const mouseDragging = ref(false);

    //Минимальный рендж по ху
    const minSquareXY = computed(() => {
      return circleSize.value / 2 + squareSize.value / 2 - (alternativeMode.value ? (squareSize.value / 2) * 2 : 0)
    })

    //Максимальный рендж по ху
    const maxSquareXY = computed(
        () => {
          return circleSize.value + minSquareXY.value - squareSize.value + (alternativeMode.value ? (squareSize.value / 2) * 2 : 0);
        }
    )

    // Функция для проверки, находится ли квадрат внутри окружности
    const checkSquarePosition = () => {
      const distance = Math.sqrt(Math.pow(squareX.value - circleSize.value, 2) + Math.pow(squareY.value - circleSize.value, 2));
      const maxDistance = (circleSize.value / 2 - squareSize.value / 2) + (alternativeMode.value ? (squareSize.value / 2) : 0);
      if (distance > maxDistance) {
        const angle = Math.atan2(squareY.value - circleSize.value, squareX.value - circleSize.value);
        squareX.value = Math.round(circleSize.value + maxDistance * Math.cos(angle));
        squareY.value = Math.round(circleSize.value + maxDistance * Math.sin(angle));
      }
    }

    // Перемещение квадрата
    const dragSquare = (event) => {
      if (mouseDragging.value) {
        // Определение координат мыши на canvas
        const mousePos = getMousePos(event, canvas.value);

        const distance = Math.sqrt(Math.pow(mousePos.x - circleSize.value, 2) + Math.pow(mousePos.y - circleSize.value, 2));

        // Проверка, находится ли точка внутри окружности
        if (distance < circleSize.value / 2) {
          // Перемещение квадрата
          squareX.value = mousePos.x;
          squareY.value = mousePos.y;
          checkSquarePosition();
          draw();
        }
      }
    }

    const draw = () => {
      ctx.value.clearRect(0, 0, canvas.value.width, canvas.value.height); // Очистка canvas

      // Рисование окружности
      ctx.value.beginPath();
      ctx.value.arc(circleSize.value, circleSize.value, circleSize.value / 2, 0, Math.PI * 2);
      ctx.value.stroke();

      // Рисование квадрата
      ctx.value.fillRect(squareX.value - squareSize.value / 2, squareY.value - squareSize.value / 2, squareSize.value, squareSize.value);
    }

    const toggleMouseDragging = (value) => {
      mouseDragging.value = value;
    }

    //Если изменен размер окружности, ставим квадрат по центру
    watch(
        () => circleSize.value,
        () => {
          setSquareOnCenter()
          setTimeout(
              () => {
                if (circleSize.value > 500) {
                  alert("Окружность не может быть больше 500");
                  circleSize.value = 500;
                  setSquareOnCenter()
                  draw();
                }
                if(circleSize.value < squareSize.value + 50) {
                  alert("Окружность не может быть меньше квадрата");
                  circleSize.value = squareSize.value + 50;
                  setSquareOnCenter()
                  draw();
                }
              }, 750
          )
        }
    )

    //Если изменен размер квадрата ставим его по центру
    watch(
        () => squareSize.value,
        () => {
          setSquareOnCenter();
          //Если квадрат не влезает в окружность
          if (squareSize.value * Math.sqrt(2) > circleSize.value - 20) {
            alert('Не, не, не, так не пойдёт');
            squareSize.value = Math.round(circleSize.value / Math.sqrt(2) - 20);
          }
        }
    )

    watch(
        () => [squareX.value, squareY.value],
        () => {
          checkSquarePosition();
        }
    )

    onMounted(
        () => {
          canvas.value = document.getElementById("myCanvas");
          ctx.value = canvas.value.getContext("2d");
          setSquareOnCenter();
          draw();
        }
    )

    return {
      canvas,
      squareSize,
      circleSize,
      toggleMouseDragging,
      dragSquare,
      draw,
      squareX,
      squareY,
      minSquareXY,
      maxSquareXY,
      alternativeMode,
      canvasHeight,
      canvasWidth,
    };
  },
};
</script>

<style>
.circle-square {
  position: relative;
  display: flex;
}
</style>
