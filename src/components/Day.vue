<template>
  <div class="day">
    <h2>☄️ Will we die {{dayId | toDay}}?</h2>
    <h3 :style="{color: warnMessage.color}">{{warnMessage.value}}</h3>
    <div class="content">
      <ul>
        <li>
          name:
          <strong>{{neo.name.replace(/[\(\)]/g, '"')}}</strong>
        </li>
        <li>
          diameter:
          ~
          <strong>{{diameter.value | round}} {{diameter.unit}}</strong>
        </li>
        <li>
          speed:
          <strong>{{speed | round}} {{shortUnit}}/h</strong>
        </li>
        <li>
          distance:
          <strong>{{missDist | round}} {{shortUnit}}</strong>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "Day",
  data() {
    return {
      neo: null,
      diameter: {
        unit: null,
        value: 0
      },
      shortUnit: "km",
      missDist: null,
      speed: 0,
      date: "",
      warnMessage: {}
    };
  },
  beforeMount: function() {
    this.getData();
    this.setData();
  },
  props: {
    data: Array,
    meters: Boolean,
    dayId: Number
  },
  filters: {
    round: function(number) {
      return Math.round(number).toLocaleString();
    },
    toDay: function(number) {
      switch (number) {
        case 0: {
          return "today";
        }
        case 1: {
          return "tomorrow";
        }
        case 2: {
          return "after tomorrow";
        }
        default: {
          return `in ${number} days`;
        }
      }
    }
  },
  watch: {
    meters: function() {
      this.setData();
    }
  },
  methods: {
    getData() {
      this.neo = this.data.sort(
        (a, b) =>
          a.close_approach_data[0].miss_distance.lunar -
          b.close_approach_data[0].miss_distance.lunar
      )[0];
    },
    setData() {
      const unitsM = [
        { name: "meters", short: "m" },
        { name: "kilometers", short: "km" }
      ];
      const unitsF = [
        { name: "feet", short: "ft" },
        { name: "miles", short: "mi" }
      ];

      const units = this.meters ? unitsM : unitsF;
      this.shortUnit = units[1].short;

      const neo_ed = this.neo.estimated_diameter;
      const unit =
        (neo_ed[units[1].name].estimated_diameter_min | 0) > 0
          ? units[1]
          : units[0];
      const value =
        (neo_ed[unit.name].estimated_diameter_min +
          neo_ed[unit.name].estimated_diameter_max) /
        2;

      this.diameter.unit = unit.short;
      this.diameter.value = value;

      const neo_cad = this.neo.close_approach_data[0];
      this.date = new Date(
        neo_cad.epoch_date_close_approach
      ).toLocaleDateString();
      this.speed = neo_cad.relative_velocity[units[1].name + "_per_hour"];
      this.missDist = neo_cad.miss_distance[units[1].name];

      const warnMessages = [
        { value: "All fine 😎", color: "green" },
        { value: "Maybe? 😅", color: "orange" },
        { value: "Nuka-Cola time! 🥤", color: "red" }
      ];

      const kmMiss = neo_cad.miss_distance.kilometers;
      const warnId = kmMiss > 1000000 ? 0 : kmMiss > 500000 ? 1 : 2;
      this.warnMessage = warnMessages[warnId];
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.day {
  /* --offset: 100; */
  display: none;
  position: relative;
  opacity: 0;
  width: 100%;
  margin: 0 auto;
  transition: all ease-in 0.5s;
  transition-property: left, opacity;
  text-align: center;
}

.left {
  display: block;
  position: absolute;
  top: 50%;
  opacity: 0;
  left: -100%;
  transform: translateY(-50%);
}

h2 {
  color: #ccc;
}

strong {
  color: #aaa;
}

.middle {
  display: block;
  opacity: 1;
  left: 0;
  /* transition-timing-function: ease-in; */
}

.right {
  display: block;
  position: absolute;
  opacity: 0;
  top: 50%;
  left: 100%;
  transform: translateY(-50%);
}

.content {
  padding: 5px 10px;
}

li {
  list-style-type: none;
}
</style>
