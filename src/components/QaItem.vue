<template>
  <div class="wrapper">
    <div class="q" @click="open = !open">
      {{q}}
      <img alt :class="open && 'rotate'" src="../assets/arrow_down.svg"/>
    </div>
    <transition name="accordion"
                @before-enter="beforeEnter"
                @enter="enter"
                @enter-cancelled="enterCancelled"
                @leave="leave"
                @after-leave="afterLeave"
                @leave-cancelled="leaveCancelled">
      <pre v-show="open" class="a" v-html="a">
      </pre>
    </transition>
  </div>
</template>

<script>

function resetStyles (el) {
  const size = el._initialStyle['height'];
  el.style.overflow = el._initialStyle.overflow;
  if (size != null) el.style['height'] = size;
  delete el._initialStyle;
}

function afterLeave (el) {
  if (el._parent) {
    el._parent.classList.remove('accordion');
  }
  resetStyles(el)
}

export default {
  name: "QaItem",
  props: ['q', 'a', 'note'],
  data() {
    return {
      open: false
    }
  },
  methods: {
    beforeEnter (el) {
      el._parent = el.parentNode;
      el._initialStyle = {
        transition: el.style.transition,
        visibility: el.style.visibility,
        overflow: el.style.overflow,
        ['height']: el.style['height'],
      };
    },
    enter (el) {
      const initialStyle = el._initialStyle;
      const offset = `${el['offsetHeight']}px`;

      el.style.setProperty('transition', 'none', 'important');
      el.style.visibility = 'hidden';
      el.style.visibility = initialStyle.visibility;
      el.style.overflow = 'hidden';
      el.style['height'] = '0';

      void el.offsetHeight; // force reflow

      el.style.transition = initialStyle.transition;

      if (el._parent) {
        el._parent.classList.add('accordion');
      }

      requestAnimationFrame(() => {
        el.style['height'] = offset
      });
    },

    afterEnter: resetStyles,
    enterCancelled: resetStyles,

    leave (el) {
      el._initialStyle = {
        transition: '',
        visibility: '',
        overflow: el.style.overflow,
        ['height']: el.style['height'],
      };

      el.style.overflow = 'hidden';
      el.style['height'] = `${el['offsetHeight']}px`;
      void el.offsetHeight // force reflow

      requestAnimationFrame(() => (el.style['height'] = '0'));
    },

    afterLeave,
    leaveCancelled: afterLeave,
  }
}
</script>

<style scoped>
  .wrapper {
    padding: 20px 0;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  }

  .q {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-weight: bold;
    font-size: 16px;
    line-height: 24px;
    cursor: pointer;
  }

  .q img {
    margin-right: 12px;
    transition: all 0.5s;
  }

  .q img.rotate {
    transform: rotate(180deg);
  }

  .a {
    margin: 8px 0 0;
    font-size: 14px;
    line-height: 24px;
    color: #D9DAF2;
    transition: all 0.15s;
    white-space: normal;
    font-family: 'Nunito', sans-serif;
  }

  .note {
    font-size: 12px;
    font-style: italic;
    color: #D9DAF290
  }
</style>