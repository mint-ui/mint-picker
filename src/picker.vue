<template>
  <div class="picker" :class="{ 'picker-3d': rotateEffect }">
    <div class="picker-toolbar" v-if="showToolbar"><slot></slot></div>
    <div class="picker-items">
      <slots v-ref:slots></slots>
      <div class="picker-center-highlight"></div>
    </div>
  </div>
</template>

<style>
  .picker {
    overflow: hidden;
  }

  .picker-toolbar {
    height: 40px;
  }

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
    background-color: #eaeaea;
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
      slots: {
        type: Array
      },
      showToolbar: {
        type: Boolean,
        default: true
      },
      visibleItemCount: {
        type: Number,
        default: 5
      },
      rotateEffect: {
        type: Boolean,
        default: false
      }
    },

    beforeCompile() {
      var slots = this.slots || [];
      var values = this.values;

      if (values.length !== slots.length) {
        values = this.values = [];
        var valueIndexCount = 0;
        slots.forEach(function(slot) {
          if (!slot.divider) {
            slot.valueIndex = valueIndexCount++;
            values[slot.valueIndex] = (slot.values || [])[0];
          }
        });
      }
    },

    methods: {
      getSlot(slotIndex) {
        var slots = this.slots || [];
        var count = 0;
        var target;
        var children = this.$refs.slots.$children;

        slots.forEach(function(slot, index) {
          if (!slot.divider) {
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
      },
      getSlotValues(index) {
        var slot = this.getSlot(index);
        if (slot) {
          return slot.values;
        }
        return null;
      },
      setSlotValues(index, values) {
        var slot = this.getSlot(index);
        if (slot) {
          slot.values = values;
        }
      },
      getValues() {
        return this.values;
      },
      setValues(values) {
        var slotCount = this.slotCount;
        values = values || [];
        if (slotCount !== values.length) {
          throw new Error('values length is not equal slot count.');
        }
        values.forEach((value, index) => {
          this.setSlotValue(index, value);
        });
      }
    },

    events: {
      slotValueChange() {
        this.$emit('change', this, this.values);
      }
    },

    computed: {
      values() {
        var slots = this.slots || [];
        var values = [];
        slots.forEach(function(slot) {
          if (!slot.divider) values.push(slot.value);
        });

        return values;
      },
      slotCount() {
        var slots = this.slots || [];
        var result = 0;
        slots.forEach(function(slot) {
          if (!slot.divider) result++;
        });
        return result;
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
          var slots = parent.slots;
          var values = parent.values;
          var template = '';
          var valueIndexCount = 0;

          slots.forEach(function(slot, index) {
            if (!slot.divider) {
              values[valueIndexCount] = (slot.values || [])[0];
              template += `<picker-slot :values="$parent.slots[${index}].values || []" :text-align="$parent.slots[${index}].textAlign || 'center'" :visible-item-count="$parent.visibleItemCount" :class-name="$parent.slots[${index}].className" :flex="$parent.slots[${index}].flex" :value.sync="$parent.values[${valueIndexCount}]"  ${ parent.rotateEffect ? 'rotate-effect' : '' } ></picker-slot>`;
              valueIndexCount++;
            } else {
              template += `<picker-slot divider :content="$parent.slots[${index}].content"></picker-slot>`;
            }
          });

          this.$options.template = template;
        }
      }
    }
  };
</script>
