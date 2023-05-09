<script setup>
  import { ref, shallowRef, computed, watch, nextTick } from 'vue'
  import Chart from 'chart.js/auto'

  const weights = ref([])

  const weightChartE1 = ref(null)

  const weightChart = shallowRef(null)

  const weightInput = ref(60.0)

  const currentWeight = computed(() => {
    return weights.value.sort((a, b) => b.date - a.date)[0] || {weight: 0}
  })

  const addWeight = () => {
    weights.value.push({
      weight: weightInput.value,
      date: new Date().getTime()
    })
  }

  watch(weights, newWeights => {
    const ws = [...newWeights]

    if (weightChart.value) {
      weightChart.value.data.labels = ws
        .sort((a, b) => a.date - b.date)
        .map(w => new Date(w.date).toLocaleDateString())
        .slice(-7)

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map(w => w.weight)
        .slice(-7)

      weightChart.value.update()

      return
    }

    nextTick(() => {
      weightChart.value = new Chart(weightChartE1.value.getContext('2d'), {
        type:'line',
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map(w => new Date(w.date).toLocaleDateString()),
          datasets: [
            {
              label: 'Weight',
              data: ws 
                .sort((a, b) => a.date - b.date)
                .map(w => w.weight),
              backgroundColor: 'rgba(255, 105, 180, 0.2)',
              borderColor: 'rgb(255, 105, 180)',
              borderWidth: 1,
              fill: true
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
        }
      })
    })
  }, {deep: true})

</script>

<template>
  <main>
    <h1>Weight Tracker</h1>

    <div class="current">
      <span>
        {{ currentWeight.weight }}
      </span>
      <small>
        Current weight (kg)
      </small>
    </div>

    <form @submit.prevent="addWeight">

      <input type="number" step="0.1" v-model="weightInput"/>

      <input type="submit" value="Add weight"/>

    </form>

    <div v-if="weights && weights.length > 0">

      <h2>Last 7 days</h2>

      <div class="canvas-box">
        <canvas ref="weightChartE1"></canvas>
      </div>

      <div class="weight-history">
        <h2>Weight history</h2>
        <ul>
          <li v-for="weight in weights">
            <span>
              {{ weight.weight }}kg
            </span>
            <small>
              {{ new Date(weight.date).toLocaleDateString() }}
            </small>
          </li>
        </ul>
      </div>

    </div>

  </main>
</template>

<style >
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'montserrat', sans-serif;
}

body{
  background-color: #eee;
}

main{
  padding: 1.5rem;
}

h1{
  font-size: 2em;
  text-align: center;
  margin-bottom: 2rem;
}

h2{
  margin-bottom: 1rem;
  color: #888;
  font-weight: 400;
}

.current{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  width: 200px;
  height: 200px;

  text-align: center;
  background-color: white;
  border-radius: 999px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  border: 5px solid hwb(330 41% 0%);

  margin: 0 auto 2rem;
}

.current span{
  display: block;
  font-size: 2em;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.current small{
  color: #888;
  font-style: italic;
}

form{
  display: flex;
  margin-bottom: 2rem;
  border: 1px solid #AAA;
  border-radius: 0.5rem;
  overflow: hidden;
  transition: 200ms linear;
}

form:focus-within,
form:hover{
  border-color: hotpink;
  border-width: 2px;
}

form input[type="number"]{
  appearance: none;
  outline: none;
  border: none;
  background-color: white;

  flex: 1 1 0%;
  padding: 1rem 1.5rem;
  font-size: 1.25rem;
}
</style>
