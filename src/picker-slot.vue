<template>
  <div class="picker-items-col picker-items-col-center" :class="{ 'picker-items-col-divider': divider, 'picker-items-col-absolute': rotateEffect }"
    :style="{ flex: flex }">
    <div v-if="!divider" v-el:wrapper class="picker-items-col-wrapper" :class="{ dragging: dragging }" :style="{ height: 36 * visibleItemCount + 'px' }">
      <div class="picker-item" v-for="itemValue in values" :class="{ 'picker-selected': itemValue === value }">{{ itemValue }}</div>
    </div>
    <div v-if="divider">{{ content }}</div>
  </div>
</template>

<script type="text/ecmascript-6" lang="babel">
  import Vue from 'vue';
  import draggable from './draggable';
  import translateUtil from './translate';

  import { addClass, removeClass, once } from 'wind-dom';

  var rotateElement = function(element, angle) {
    if (!element) return;
    var transformProperty = translateUtil.transformProperty;

    element.style[transformProperty] = element.style[transformProperty].replace(/rotateX\(.+?deg\)/gi, '') + ` rotateX(${angle}deg)`;
  };

  require('raf.js');

  export default {
    props: {
      values: {
        type: Array,
        default() {
          return [];
        }
      },
      value: {},
      visibleItemCount: {
        type: Number,
        default: 5
      },
      rotateEffect: {
        type: Boolean,
        default: false
      },
      divider: {
        type: Boolean,
        default: false
      },
      flex: {},
      width: {},
      className: {},
      content: {}
    },

    data() {
      return {
        dragging: false,
        animationFrameId: null
      };
    },
    
    computed: {
      valueIndex() {
        return this.values.indexOf(this.value);
      },
      dragRange() {
        var values = this.values;
        var visibleItemCount = this.visibleItemCount;
        
        return [ -36 * (values.length - Math.ceil(visibleItemCount / 2)), 36 * Math.floor(visibleItemCount / 2) ];
      }
    },

    methods: {
      value2Translate(value) {
        var values = this.values;
        var valueIndex = values.indexOf(value);
        var offset = Math.floor(this.visibleItemCount / 2);

        if (valueIndex !== -1) {
          return (valueIndex - offset) * -36;
        }
      },

      translate2Value(translate) {
        translate = Math.round(translate / 36) * 36;
        var index = -(translate - Math.floor(this.visibleItemCount / 2) * 36) / 36;

        return this.values[index];
      },

      updateRotate: function (currentTranslate, pickerItems) {
        var dragRange = this.dragRange;
        var wrapper = this.$els.wrapper;

        if (!pickerItems) {
          pickerItems = wrapper.querySelectorAll('.picker-item');
        }

        if (currentTranslate === undefined) {
          currentTranslate = translateUtil.getElementTranslate(wrapper).top;
        }

        [].forEach.call(pickerItems, (item, index) => {
          var itemOffsetTop = index * 36;
          var translateOffset = dragRange[1] - currentTranslate;
          var itemOffset = itemOffsetTop - translateOffset;
          var percentage = itemOffset / 36;

          var itemsFit = Math.ceil(this.visibleItemCount / 2);

          var angle = (-20 * percentage);
          if (angle > 180) angle = 180;
          if (angle < -180) angle = -180;

          rotateElement(item, angle);

          if (Math.abs(percentage) > itemsFit) {
            addClass(item, 'picker-item-far');
          } else {
            removeClass(item, 'picker-item-far');
          }
        });
      },

      initEvents() {
        var el = this.$els.wrapper;
        var dragState = {};

        var velocityTranslate, velocityTime, prevTranslate, pickerItems;

        draggable(el, {
          start: (event) => {
            cancelAnimationFrame(this.animationFrameId);
            dragState = {
              range: this.dragRange,
              start: new Date(),
              startLeft: event.pageX,
              startTop: event.pageY,
              startTranslateTop: translateUtil.getElementTranslate(el).top
            };
            pickerItems = el.querySelectorAll('.picker-item');
          },

          drag: (event) => {
            this.dragging = true;

            dragState.left = event.pageX;
            dragState.top = event.pageY;

            var deltaY = dragState.top - dragState.startTop;
            var translate = dragState.startTranslateTop + deltaY;

            translateUtil.translateElement(el, null, translate);

            velocityTranslate = translate - prevTranslate || translate;
            velocityTime = Date.now();

            prevTranslate = translate;

            if (this.rotateEffect) {
              this.updateRotate(prevTranslate, pickerItems);
            }
          },

          end: () => {
            this.dragging = false;

            var momentumRatio = 7;

            var currentTranslate = translateUtil.getElementTranslate(el).top;
            var duration = new Date() - dragState.start;

            var momentumTranslate;
            if (duration < 300) {
              momentumTranslate = currentTranslate + velocityTranslate * momentumRatio;
              momentumTranslate = Math.max(Math.min(momentumTranslate, dragState.range[1]), dragState.range[0]);
            }

            dragState = {};

            Vue.nextTick(() => {
              var translate;
              if (momentumTranslate) {
                translate = Math.round(momentumTranslate / 36) * 36;
              } else {
                translate = Math.round(currentTranslate / 36) * 36;
              }

              if (this.rotateEffect) {
                this.animationFrameId = requestAnimationFrame(() => {
                  this.updateRotate();
                });

                once(el, translateUtil.transitionEndProperty, () => {
                  cancelAnimationFrame(this.animationFrameId);
                });
              }

              translateUtil.translateElement(el, null, translate);

              this.value = this.translate2Value(translate);
            });
          }
        });
      },

      doOnValueChange() {
        var value = this.value;
        var wrapper = this.$els.wrapper;

        translateUtil.translateElement(wrapper, null, this.value2Translate(value));
      },

      doOnValuesChange() {
        var el = this.$el;
        var items = el.querySelectorAll('.picker-item');
        [].forEach.call(items, (item, index) => {
          translateUtil.translateElement(item, null, 36 * index);
        });
      }
    },

    ready() {
      this.ready = true;

      if (!this.divider) {
        this.initEvents();
        this.doOnValueChange();
      }
      if (this.rotateEffect) {
        this.doOnValuesChange();
      }
    },

    watch: {
      values() {
        if (this.rotateEffect) {
          this.doOnValuesChange();
        }
      },
      value(newVal) {
        this.doOnValueChange();

        var valueIndex = this.valueIndex;

        var items = this.$els.wrapper.querySelectorAll('.picker-item');


        this.$dispatch('slotValueChange', this);
      }
    }
  };
</script>
