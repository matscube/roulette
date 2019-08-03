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
      rouletteLock: false,
      position: 0,

      // Apperance Param
      radius: 400,
      centerRadius: 150,
      blinkCount: 5,
      blinkDuration: 100, // millsec

      // Shuffle Param
      totalTime: 5, // sec
      baseShuffleCount: 200,
      speedAlloc: [10, 9, 8, 7, 6, 5, 4, 3, 2, 1],
      timeAlloc: [1, 1, 1, 1, 1, 1, 1, 2, 2, 3],
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
      let active = this.position == i
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
    createShuffle: function() {
      // Calc shuffle/duration allocation from fixed totalTime and shuffleCount

      let shuffleCount = this.baseShuffleCount + this.randomInt(this.count)
      let timeAllocSum = this.timeAlloc.reduce((sum, n) => sum += n, 0)
      let st = this.speedAlloc.map((s, i) => s * this.timeAlloc[i])
      let stSum = st.reduce((sum, n) => sum += n, 0)
      let speedAlpha = timeAllocSum / this.totalTime * shuffleCount / stSum
      let durationAlloc = this.speedAlloc.map((d) => 1 / speedAlpha / d * 1000)
      
      let shuffleAlloc = st.map((s) => shuffleCount / stSum * s)
      var shuffleThreshold = []
      shuffleAlloc.reverse().reduce(function(sum, n){
        sum += n
        shuffleThreshold.push(sum)
        return sum
      }, 0)
      shuffleThreshold = shuffleThreshold.reverse()
      
      return {
        'startPosition': this.randomInt(this.count),
        'shuffleCount': shuffleCount,
        'shuffleThreshold': shuffleThreshold,
        'durationAlloc': durationAlloc,
      }
    },
    shuffle: function() {
      let shuffleParams = this.createShuffle()
      var shuffleRemain = shuffleParams.shuffleCount
      this.position = shuffleParams.startPosition
      var timer = null;
      var self = this;
      var forwardRoulette = function() {
        self.options[self.position].active = false
        self.position = (self.position + 1) % self.count
        self.options[self.position].active = true
      }
      var loopRoulette = function() {
        forwardRoulette()
        if (--shuffleRemain > 0) {
          var duration;
          for (var i = shuffleParams.shuffleThreshold.length - 1; i >= 0; i--) {
            if (shuffleRemain < shuffleParams.shuffleThreshold[i]) {
              duration = shuffleParams.durationAlloc[i]
              break
            }
          }
          setTimeout(loopRoulette, duration);
        } else {
          clearTimeout(timer);
          self.finished()
        }
      }
      timer = setTimeout(loopRoulette, 0);
    },
    start: function() {
      if (this.rouletteLock) {
        return
      } 
      this.readyStart()
      this.shuffle()
    },
    readyStart: function() {
      this.rouletteLock = true
      for (const option of this.options) {
        option.active = false
      }
    },
    finished: function() {
      this.blink(() => {
        this.rouletteLock = false
      })
    },
    blink: function(callback) {
      var timer = null;
      var self = this;
      var countRemain = this.blinkCount * 2 // must be even number
      var toggleActive = function() {
        if (countRemain-- > 0) {
          self.options[self.position].active = !self.options[self.position].active
          setTimeout(toggleActive, self.blinkDuration);
        } else {
          clearTimeout(timer);
          callback()
        }
      }
      timer = setTimeout(toggleActive, 0);
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
