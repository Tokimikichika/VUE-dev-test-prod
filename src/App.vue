<template>
  <div id="app">
    <div class="header">
      <h1>Количество пройденных тестов: "{{ data?.title || 'Выберите данные' }}"</h1>
    </div>
    <div>
      <label>Выберите набор данных:</label>
      <select v-model="selectedFile" @change="loadData">
        <option value="https://rcslabs.ru/ttrp1.json">Данные 1</option>
        <option value="https://rcslabs.ru/ttrp2.json">Данные 2</option>
        <option value="https://rcslabs.ru/ttrp3.json">Данные 3</option>
        <option value="https://rcslabs.ru/ttrp4.json">Данные 4</option>
        <option value="https://rcslabs.ru/ttrp5.json">Данные 5</option>
      </select>
    </div>
    <TestVisualization :data="data" />
  </div>
</template>

<script>
import TestVisualization from "./components/TestVisualization.vue";

export default {
  components: {
    TestVisualization,
  },
  data() {
    return {
      selectedFile: "https://rcslabs.ru/ttrp1.json",
      data: null,
    };
  },
  methods: {
    async loadData() {
      try {
        const response = await fetch(this.selectedFile);
        this.data = await response.json();
      } catch (error) {
        console.error("Ошибка загрузки данных:", error);
      }
    },
  },
  mounted() {
    this.loadData();
  },
};
</script>

<style>
#app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

.header {
  text-align: right;
  margin-bottom: 10px;
}
</style>
