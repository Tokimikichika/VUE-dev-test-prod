<template>
  <div v-if="data" class="visualization">
    <div class="bars">
      <div class="instance" v-for="(instance, key) in instances" :key="key">
        <div class="bar front" :style="{ height: `${normalizedHeight(instance.front)}px` }">
          <span class="value">{{ instance.front }}</span>
        </div>
        <div class="bar back" :style="{ height: `${normalizedHeight(instance.back)}px` }">
          <span class="value">{{ instance.back }}</span>
        </div>
        <div class="bar db" :style="{ height: `${normalizedHeight(instance.db)}px` }">
          <span class="value">{{ instance.db }}</span>
        </div>
        <div class="label">{{ key }}</div>
      </div>
      <div class="norm-instance">
        <div class="bar norm" :style="{ height: `${normalizedHeight(data.norm)}px` }">
          <div class="stripes"></div>
          <div class="norm-value">{{ data.norm }}</div>
        </div>
        <div class="label">норматив</div>
      </div>
    </div>
    
    <div class="arrows">
      <div
        class="arrow-container"
        v-for="(offset, key) in offsets"
        :key="key"
        :style="arrowPositionStyle(key)"
      >
        <svg class="arrow" :class="{ positive: offset >= 0, negative: offset < 0 }" width="24" height="24">
          <path
            :d="offset >= 0 ? 'M12 19V5M5 12l7-7 7 7' : 'M12 5v14M5 12l7 7 7-7'"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          />
        </svg>
        <span :class="{ positive: offset >= 0, negative: offset < 0 }">
          {{ offset > 0 ? `+${offset}` : offset }}
        </span>
      </div>
      
      <div v-for="(key, index) in ['dev-test', 'test-prod']" :key="index" class="p-arrow" :style="pArrowPositionStyle(key)">
        <svg width="100" height="100">
          <path d="M10 60 V10 H90 V60" stroke="gray" stroke-width="2" fill="none" marker-end="url(#arrowhead)" />
          <defs>
            <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="10" refY="3.5" orient="auto">
              <polygon points="0 0, 10 3.5, 0 7" fill="gray" />
            </marker>
          </defs>
        </svg>
      </div>
    </div>
    
    <div class="legend">
      <div class="legend-item">
        <div class="color-box front"></div> Клиентская часть
      </div>
      <div class="legend-item">
        <div class="color-box back"></div> Серверная часть
      </div>
      <div class="legend-item">
        <div class="color-box db"></div> База данных
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    data: {
      type: Object,
      required: true,
    },
  },
  computed: {
    instances() {
      return {
        dev: this.data.dev,
        test: this.data.test,
        prod: this.data.prod,
      };
    },
    offsets() {
      return {
        "dev-test": this.total(this.data.test) - this.total(this.data.dev),
        "test-prod": this.total(this.data.prod) - this.total(this.data.test),
      };
    },
    maxInstanceValue() {
      return Math.max(
        this.data.norm,
        ...Object.values(this.instances).flatMap(instance => [instance.front, instance.back, instance.db])
      );
    },
  },
  methods: {
    total(instance) {
      return instance.front + instance.back + instance.db;
    },
    normalizedHeight(value) {
      const maxHeight = 90; 
      if (this.maxInstanceValue === 0) return 0;
      return (value / this.maxInstanceValue) * maxHeight;
    },
    arrowPositionStyle(key) {
      const heights = {
        "dev-test": this.total(this.data.dev),
        "test-prod": this.total(this.data.test),
      };

      const baseTop = 150 - this.normalizedHeight(heights[key]);
      let offsetTop = baseTop;

      if (key === "test-prod" && heights["test-prod"] >= heights["dev-test"]) {
        offsetTop += 30; 
      }

      return {
        position: 'absolute',
        left: key === 'dev-test' ? '40%' : '50%',
        top: `${offsetTop}px`,
        transform: 'translateX(-50%)',
      };
    },
    pArrowPositionStyle(key) {
      const heights = {
        "dev-test": this.total(this.data.dev),
        "test-prod": this.total(this.data.test),
      };

      const baseTop = 170 - this.normalizedHeight(heights[key]);
      let offsetTop = baseTop;

      if (key === "test-prod" && heights["test-prod"] >= heights["dev-test"]) {
        offsetTop += 20;
      }

      return {
        position: 'absolute',
        left: key === 'dev-test' ? '40%' : '50%',
        top: `${offsetTop}px`,
        transform: 'translateX(-50%)',
      };
    },
  },
};
</script>

<style scoped>
.visualization {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}

.bars {
  display: flex;
  align-items: flex-end;
  height: 300px;
  margin-bottom: 20px;
  position: relative;
}

.instance, .norm-instance {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0 15px;
}

.bar {
  width: 40px;
  position: relative;
  margin-bottom: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.front {
  background-color: #66b2ff;
}

.back {
  background-color: #a855f7;
}

.db {
  background-color: #fb7185;
}

.norm {
  background: repeating-linear-gradient(
    120deg,
    #66b2ff,
    #66b2ff 15px,
    #ffffff 10px,
    #ffffff 20px
  );
  position: relative;
}

.value {
  position: absolute;
  color: #fff;
  font-weight: bold;
}

.norm-value {
  background-color: #ffffff;
  color: #000000;
  padding: 2px 4px;
  border-radius: 3px;
  position: absolute;
}

.label {
  margin-top: 5px;
  text-align: center;
  font-weight: 500;
}

.arrows {
  width: 100%;
  margin-top: 10px;
  position: absolute;
  top: 0;
}

.arrow-container {
  display: flex;
  align-items: center;
  flex-direction: column;
  justify-content: center;
}

.arrow {
  font-size: 18px;
  margin-bottom: 4px;
}

.p-arrow {
  position: absolute;
  width: 100px;
  height: 100px;
}

.positive {
  color: green;
  background-color: #f1f1f1;
  padding: 3px 6px;
  border-radius: 5px;
}

.negative {
  color: red;
  background-color: #f1f1f1;
  padding: 3px 6px;
  border-radius: 5px;
}

.stripes {
  background: repeating-linear-gradient(
    120deg,
    #66b2ff,
    #66b2ff 15px,
    #ffffff 10px,
    #ffffff 20px
  );
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0.2;
}

.legend {
  display: flex;
  margin-top: 20px;
}

.legend-item {
  display: flex;
  align-items: center;
  margin-right: 15px;
}

.color-box {
  width: 20px;
  height: 20px;
  margin-right: 8px;
}

.color-box.front {
  background-color: #66b2ff;
}

.color-box.back {
  background-color: #a855f7;
}

.color-box.db {
  background-color: #fb7185;
}
</style>
