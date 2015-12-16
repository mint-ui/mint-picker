<template>
  <div class="picker picker-3d" style="overflow: hidden;">
    <div class="picker-model-inner picker-items">
      <slots v-ref:slots></slots>
      <div class="picker-center-highlight"></div>
    </div>
  </div>
</template>

<style>
  .picker-items {
    display: -webkit-box;
    display: -ms-flexbox;
    display: -webkit-flex;
    display: flex;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    -webkit-justify-content: center;
    justify-content: center;
    padding: 0;
    text-align: right;
    font-size: 24px;
  }

  .picker-items-col {
    overflow: hidden;
    position: relative;
    max-height: 100%
  }

  .picker-items-col.picker-items-col-left {
    text-align: left;
  }

  .picker-items-col.picker-items-col-center {
    text-align: center;
  }

  .picker-items-col.picker-items-col-right {
    text-align: right;
  }

  .picker-items-col.picker-items-col-divider {
    color: #000;
    display: -webkit-box;
    display: -ms-flexbox;
    display: -webkit-flex;
    display: flex;
    -webkit-box-align: center;
    -ms-flex-align: center;
    -webkit-align-items: center;
    align-items: center
  }

  .picker-items-col-wrapper {
    -webkit-transition-duration: 0.3s;
    transition-duration: 0.3s;
    -webkit-transition-timing-function: ease-out;
    transition-timing-function: ease-out
  }

  .picker-items-col-wrapper.dragging,
  .picker-items-col-wrapper.dragging .picker-item {
    -webkit-transition-duration: 0s;
    transition-duration: 0s;
  }

  .picker-item {
    height: 36px;
    line-height: 36px;
    padding: 0 10px;
    white-space: nowrap;
    position: relative;
    overflow: hidden;
    text-overflow: ellipsis;
    color: #707274;
    left: 0;
    top: 0;
    width: 100%;
    box-sizing: border-box;
    -webkit-transition-duration: .3s;
    transition-duration: .3s
  }

  .picker-items-col-absolute .picker-item {
    position: absolute;
  }

  .picker-item.picker-item-far {
    pointer-events: none
  }

  .picker-item.picker-selected {
    color: #000;
    -webkit-transform: translate3d(0, 0, 0) rotateX(0);
    transform: translate3d(0, 0, 0) rotateX(0);
  }

  .picker-3d .picker-items {
    overflow: hidden;
    -webkit-perspective: 1200px;
    perspective: 1200px
  }

  .picker-3d .picker-item,
  .picker-3d .picker-items-col,
  .picker-3d .picker-items-col-wrapper {
    -webkit-transform-style: preserve-3d;
    transform-style: preserve-3d
  }

  .picker-3d .picker-items-col {
    overflow: visible
  }

  .picker-3d .picker-item {
    -webkit-transform-origin: center center;
    transform-origin: center center;
    -webkit-backface-visibility: hidden;
    backface-visibility: hidden;
    -webkit-transition-timing-function: ease-out;
    transition-timing-function: ease-out
  }

  .picker-center-highlight {
    height: 36px;
    box-sizing: border-box;
    position: absolute;
    left: 0;
    width: 100%;
    top: 50%;
    margin-top: -18px;
    pointer-events: none
  }

  .picker-center-highlight:before,
  .picker-center-highlight:after {
    content: '';
    position: absolute;
    height: 1px;
    width: 100%;
    background-color: #a8abb0;
    display: block;
    z-index: 15;
    -webkit-transform: scaleY(.5);
    transform: scaleY(0.5);
  }

  .picker-center-highlight:before {
    left: 0;
    top: 0;
    bottom: auto;
    right: auto;
  }

  .picker-center-highlight:after {
    left: 0;
    bottom: 0;
    right: auto;
    top: auto;
  }
</style>

<script type="text/ecmascript-6" lang="babel">
  export default {
    props: {
      cols: {
        type: Array
      },
      values: {
        default() {
          return [];
        }
      },
      rotateEffect: {
        type: Boolean,
        default: false
      }
    },

    beforeCompile() {
      var cols = this.cols || [];
      var values = this.values;

      if (values.length !== cols.length) {
        values = this.values = [];
        var valueIndexCount = 0;
        cols.forEach(function(col) {
          if (!col.divider) {
            col.valueIndex = valueIndexCount++;
            values[col.valueIndex] = (col.values || [])[0];
          }
        });
      }
    },

    data() {
      return {};
    },

    methods: {
      getSlot(slotIndex) {
        var cols = this.cols || [];
        var count = 0;
        var target;
        var children = this.$refs.slots.$children;

        cols.forEach(function(col, index) {
          if (!col.divider) {
            if (slotIndex === count) {
              target = children[index];
            }
            count++;
          }
        });

        return target;
      },
      getSlotValue(index) {
        var slot = this.getSlot(index);
        if (slot) {
          return slot.value;
        }
        return null;
      },
      setSlotValue(index, value) {
        var slot = this.getSlot(index);
        if (slot) {
          slot.value = value;
        }
      }
    },

    events: {
      slotValueChange() {
        this.$emit('valueschange', this, this.values);
      }
    },

    computed: {
      values() {
        var cols = this.cols || [];
        var values = [];
        cols.forEach(function(col) {
          if (!col.divider) values.push(col.value);
        });

        return values;
      }
    },

    components: {
      Slots: {
        template: '',
        components: {
          PickerSlot: require('./picker-slot.vue')
        },
        created() {
          var parent = this.$parent;
          var cols = parent.cols;
          var values = parent.values;
          var template = '';
          var valueIndexCount = 0;

          cols.forEach(function(col, index) {
            if (!col.divider) {
              values[valueIndexCount] = (col.values || [])[0];
              template += `<picker-slot :values="$parent.cols[${index}].values || []" :flex="$parent.cols[${index}].flex" ${ parent.rotateEffect ? 'rotate-effect' : '' } :value.sync="$parent.values[${valueIndexCount}]"></picker-slot>`;
              valueIndexCount++;
            } else {
              template += `<picker-slot divider :content="$parent.cols[${index}].content"></picker-slot>`;
            }
          });

          this.$options.template = template;
        }
      }
    }
  };
</script>
