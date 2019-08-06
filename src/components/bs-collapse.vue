<template>
  <div
    v-bind:class="{ collapse, collapsing, show: showCollapse }"
    v-bind:style="styleObject"
  >
    <slot></slot>
  </div>
</template>

<script>
import _ from 'lodash';

function transformTransitionDuration(stringDuration) {
  // always get the first duration (same as bootstrap)
  const durationArr = _.split(stringDuration, ',');
  const timeInSeconds = _.toNumber(_.trimEnd(_.toLower(durationArr[0]), 's'));
  if (_.isNaN(timeInSeconds)) {
    return 0;
  }
  return timeInSeconds * 1000;
}

// modified from bootstrap file bootstrap.bundle.js
function getTransitionDurationFromElement(element) {
  if (!element) {
    return 0;
  }
  const elementStyle = window.getComputedStyle(element, null);
  const animationDuration = elementStyle.getPropertyValue('animation-duration');
  const transitionDuration = elementStyle.getPropertyValue('transition-duration');
  const durationArr = [];
  durationArr.push(transformTransitionDuration(animationDuration));
  durationArr.push(transformTransitionDuration(transitionDuration));
  return _.max(durationArr);
}


function handleShowChange(newShowValue) {
  if (newShowValue) {
    this.handleShowCollapse();
  } else {
    this.handleHideCollapse();
  }
}

function handleShowCollapse() {
  if (this.collapsing) {
    this.doubleCheckState();
    return;
  }
  if (this.showCollapse) {
    return;
  }
  this.$emit('show');
  const dimension = this.getDimension();
  const scrollSize = `scroll${_.upperFirst(dimension)}`;
  this.collapsing = true;
  this.collapse = false;
  this.waitForTransitionEnd(this.$el, () => {
    this.collapsing = false;
    this.collapse = true;
    this.showCollapse = true;
    this.styleObject[dimension] = '';
    this.$emit('shown');
  });
  this.$nextTick(() => {
    this.styleObject[dimension] = `${this.$el[scrollSize]}px`;
  });
}

// based on function from David Walsh: http://davidwalsh.name/css-animation-callback
function whichTransitionEvent(el) {
  const transitionCollection = [
    { name: 'transition', event: 'transitionend' },
    { name: 'OTransition', event: 'oTransitionEnd' },
    { name: 'MozTransition', event: 'transitionend' },
    { name: 'WebkitTransition', event: 'webkitTransitionEnd' },
  ];
  for (let j = 0; j < transitionCollection.length; j += 1) {
    const styleName = transitionCollection[j].name;
    if (el.style[styleName] !== undefined) {
      return transitionCollection[j].event;
    }
  }
  return 'transitionend';
}

function waitForTransitionEnd(el, callback) {
  let transitionOver = false;
  this.$nextTick(() => {
    const transitionEventName = this.whichTransitionEvent(this.$el);
    el.addEventListener(transitionEventName, (event) => {
      if (transitionOver === false) {
        callback(event);
      }
      transitionOver = true;
    }, { once: true });
    const collapsingTransitionDuration = this.getTransitionDurationFromElement(this.$el);
    setTimeout(() => {
      if (transitionOver === false) {
        callback();
      }
      transitionOver = true;
    }, collapsingTransitionDuration + _.round(collapsingTransitionDuration / 20)); // timeout waiting after an extra 5% over due time
  });
}

function handleHideCollapse() {
  if (this.collapsing) {
    this.doubleCheckState();
    return;
  }
  if (!this.showCollapse) {
    return;
  }
  this.$emit('hide');
  const dimension = this.getDimension();
  this.styleObject[dimension] = `${this.$el.getBoundingClientRect()[dimension]}px`;
  this.collapsing = true;
  this.collapse = false;
  this.showCollapse = false;
  this.$nextTick(() => {
    this.styleObject[dimension] = '';
  });
  this.waitForTransitionEnd(this.$el, () => {
    this.collapsing = false;
    this.collapse = true;
    this.$emit('hidden');
  });
}

function getDimension() {
  if (this.width) {
    return 'width';
  }
  return 'height';
}

function doubleCheckState() {
  if (this.alreadyDoingDoubleCheck === true) {
    return;
  }
  this.alreadyDoingDoubleCheck = true;
  const collapsingTransitionDuration = this.getTransitionDurationFromElement(this.$el);
  setTimeout(() => {
    if (this.show === true && this.showCollapse === false) {
      this.handleShowCollapse();
    } else if (this.show === false && this.showCollapse === true) {
      this.handleHideCollapse();
    }
    this.alreadyDoingDoubleCheck = false;
  }, collapsingTransitionDuration);
}

export default {
  name: 'bs-collapse',
  props: {
    show: Boolean,
    height: {
      type: Boolean,
      default: true,
    },
    width: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      collapse: true,
      collapsing: false,
      showCollapse: false,
      styleObject: {
        width: '',
        height: '',
      },
      alreadyDoingDoubleCheck: false,
    };
  },
  watch: {
    show: handleShowChange,
  },
  methods: {
    handleShowCollapse,
    handleHideCollapse,
    getDimension,
    whichTransitionEvent,
    waitForTransitionEnd,
    getTransitionDurationFromElement,
    doubleCheckState,
  },
};
</script>
