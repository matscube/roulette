<template>
  <div class="roulette" v-bind:style="rouletteStyle">
    <div class="circle">
      <template v-for="(option, index) in options">
        <div class="item"
            v-bind:style="[optionStyles[index].itemStyle, optionStyles[index].itemColorStyle]">
        </div>
        <div class="item marker"
            v-bind:class="{ active: option.active }"
            v-bind:style="optionStyles[index].itemStyle">
        </div>
        <div class="text-box" v-bind:style="optionStyles[index].textBoxStyle">
          <div class="content" v-bind:style="optionStyles[index].textContentStyle">
            {{ option.text }}
          </div>
        </div>
      </template>
    </div>
    <div class="center"
        v-on:click="start"
        v-bind:style="coreStyle">
    </div>
  </div>
</template>

<script>
export default {
  name: 'Roulette',
  props: {
    count: Number,
    texts: Array,
    colors: Array,
  },
  data: function() {
    return {
      options: [],
      radius: 400,
      centerRadius: 150,
    }
  },
  computed: {
    diameter: function() {
      return this.radius * 2
    },
    centerDiameter: function() {
      return this.centerRadius * 2
    },
    itemAngle: function() {
      return Math.PI / this.count
    },
    itemSin: function() {
      return Math.sin(this.itemAngle)
    },
    itemCos: function() {
      return Math.cos(this.itemAngle)
    },
    itemTan: function() {
      return Math.tan(this.itemAngle)
    },
    rouletteStyle: function() {
      return {
        'margin-top': -this.radius + 'px',
        'margin-left': -this.radius + 'px',
        'width': this.diameter + 'px',
        'height': this.diameter + 'px',
      }
    },
    coreStyle: function() {
      return {
        width: this.centerDiameter + 'px',
        height: this.centerDiameter + 'px',
        'margin-left': 'calc(50% - ' + this.centerRadius + 'px)',
        'margin-top': 'calc(50% - ' + this.centerRadius + 'px)',
      }
    },
    optionStyles: function() {
      var optionsStyles = []
      
      // Cut item from rectangle composed of border top / bottom / left / right
      let rectagnleHeight = this.diameter
      let rectagnleWidth = rectagnleHeight * this.itemTan
      let rectangleMarginLeft = this.radius - rectagnleWidth / 2
      let rectangleMarginTop = this.radius - rectagnleHeight / 2
      let textBoxWidth = this.radius * 3 / 4 * this.itemSin * 2
      for (var i = 0; i < this.count; i++) {
        let deg = 360 / this.count * (i)
        let color = this.colors[i % this.colors.length]
        optionsStyles.push({
          key: i,
          itemStyle: {
            'transform': 'scale(1) rotate(' + deg + 'deg)',
            'margin-left': rectangleMarginLeft + 'px',
            'margin-top': rectangleMarginTop + 'px',
            'border-top-width': rectagnleHeight / 2 + 'px',
            'border-left-width': rectagnleWidth / 2 + 'px',
            'border-bottom-width': rectagnleHeight / 2 + 'px',
            'border-right-width': rectagnleWidth / 2 + 'px',
          },
          itemColorStyle: {
            'border-top-color': color,
          },
          textBoxStyle: {
            width: textBoxWidth + 'px',
            height: this.diameter + 'px',
            'margin-left': (this.radius - textBoxWidth / 2) + 'px',
            'transform': 'rotate(' + deg + 'deg)',
          },
          textContentStyle: {
            'margin-top': this.radius - this.radius * this.itemCos + 'px',
            height: this.radius / 4 * this.itemCos +  'px',
          },
        })
      }
      return optionsStyles
    },
  },
  created: function() {
    
    if (this.count < 3) {
      console.error('count is too small')
      return
    }
    
    for (var i = 0; i < this.count; i++) {
      let active = Math.random() < 0.5
      let text = this.texts[i % this.texts.length]
      this.options.push({
        key: i,
        active: active,
        text: text,
      })
    }
  },
  methods: {
    // return 0 ~ n-1
    randomInt: function(n) {
      return Math.floor(Math.random() * Math.floor(n));
    },
    start: function() {
      console.log("start")
      this.readyStart()
      
      let startPosition = this.randomInt(this.count)
      let timeUnit = 30
      var shuffleRemain = 100 + this.randomInt(50)
      var position = startPosition
      let refreshId = setInterval(() => {
        this.options[position].active = false
        position = (position + 1) % this.count
        this.options[position].active = true
        console.log(position)
        if (--shuffleRemain <= 0) {
          clearInterval(refreshId)
        }
      }, timeUnit)
    },
    readyStart: function() {
      for (const option of this.options) {
        option.active = false
      }
    },
  }
}
</script>

<style lang="scss" scoped>

.roulette {
  position: absolute;
  top: 50%;
  left: 50%;

  .center {
    position: absolute;
    border-radius: 50%;
    background-color: white;
    z-index: 100;
  }
  .circle {
    position: absolute;

    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: #61B237;
    overflow: hidden;
  }
  .text-box {
    position: absolute;
    font-weight: bold;
    color: white;
    .content {
      font-size: 2rem;
    }
  }
  .item {
    position: absolute;
    width: 0;
    height: 0;

    border-top-style: solid;
    border-left-style: solid;
    border-bottom-style: solid;
    border-right-style: solid;

    border-left-color: transparent;
    border-bottom-color: transparent;
    border-right-color: transparent;

    &.marker {
      border-top-color: grey;
      opacity: 0.7;
      z-index: 10;
      &.active {
        opacity: 0;
      }
    }
  }
}
</style>
