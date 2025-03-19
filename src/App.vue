<template>
  <div id="app">
    <h1>Optimistic NEO weather</h1>
    <div v-if="loading" class="center">Loading...</div>
    <div v-else-if="!loading && error" class="center" style="color: red">{{error}}</div>
    <div v-else class="container">
      <SwitchUnit :value="meters" @switched="toggleType"/>
      <button class="btn btn-left" :disabled="day <= 0" @click="day -= 1">
        <svg
          viewBox="0 0 24 24"
          width="80"
          height="80"
          stroke="currentColor"
          stroke-width="2"
          fill="none"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <polyline points="15 18 9 12 15 6"></polyline>
        </svg>
      </button>
      <div id="days">
        <Day
          v-for="key in keys"
          :class="classObject(key)"
          :key="data[key].id"
          :meters="meters"
          :data="data[key]"
          :dayId="keys.indexOf(key)"
        />
      </div>
      <button class="btn btn-right" :disabled="day >= keys.length-1" @click="day += 1">
        <svg
          viewBox="0 0 24 24"
          width="80"
          height="80"
          stroke="currentColor"
          stroke-width="2"
          fill="none"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <polyline points="9 18 15 12 9 6"></polyline>
        </svg>
      </button>
    </div>
  </div>
</template> 

<script>
import Day from "./components/Day";
import SwitchUnit from "./components/Switch";

export default {
  name: "App",
  data() {
    return {
      day: 0,
      dev: false,
      data: {},
      keys: [],
      error: null,
      loading: false,
      meters: true
    };
  },
  beforeMount() {
    this.fetchData("https://api.nasa.gov/neo/rest/v1/feed?api_key=DEMO_KEY");
  },
  components: {
    Day,
    SwitchUnit
  },
  methods: {
    classObject(key) {
      const id = this.keys.indexOf(key) - this.day;
      return {
        left: id === -1,
        middle: id === 0,
        right: id === 1
      };
    },
    fetchData(url) {
      this.data = this.keys = this.error = null;
      this.loading = true;

      if (this.dev) {
        import("./data.json")
          .then(this.setData)
          .finally(() => {
            this.loading = false;
          });
      } else {
        fetch(url)
          .then(res => {
            if (!res.ok) {
              console.error(res);
              throw Error(res.statusText);
            }
            return res.json();
          })
          .then(this.setData)
          .catch(err => {
            this.error = err;
          })
          .finally(() => {
            this.loading = false;
          });
      }
    },
    setData(data) {
      const neo = data.near_earth_objects;
      const dataKeys = Object.keys(neo);
      const keys = dataKeys.sort(
        (a, b) => Number(a.replace(/-/g, "")) - Number(b.replace(/-/g, ""))
      );
      this.data = neo;
      this.keys = keys;
    },
    toggleType() {
      this.meters = !this.meters;
    }
  }
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #999;
  background: #222;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
}

h1 {
  text-align: center;
  margin: .5em;
  color: #ccc;
  font-size: 1.7em;
  font-family:Verdana, Geneva, Tahoma, sans-serif
}

p,
span,
li,
strong,
small,
span {
  line-height: 120%;
}

#app {
  width: 100%;
  height: 100%;
}

.center {
  display: flex;
  width: 100%;
  height: 100%;
  justify-content: center;
  align-items: center;
}

.container {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  width: 100%;
  height: 100%;
  max-width: 760px;
  margin: 0 auto;
}

#days {
  position: relative;
  flex: 1;
}

.btn {
  border: none;
  background: none;
  color: #777;
  transition: color ease 0.1s;
  z-index: 99;
}

.btn:hover {
  color: #999;
  cursor: pointer;
}

.btn:disabled {
  color: #444;
}
</style>
