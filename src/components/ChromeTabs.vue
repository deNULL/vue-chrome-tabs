<template>
  <div class="chrome-tabs" :data-chrome-tabs-instance-id="instanceId">
    <div class="chrome-tabs-content" ref="content">
      <div v-for="(tab, tabIndex) in tabs"
          :key="tab.id"
          ref="tabs"
          class="chrome-tab" 
          :class="{
            'chrome-tab-current': (tab.id === currentTab),
          }"
          :style="{
            width: tabWidth + 'px',
            zIndex: (tab.id === currentTab) ? (tabs.length + 1) : (tabs.length - tabIndex),
          }"
          @click="currentTab = tab.id">
        <div class="chrome-tab-background">
          <svg version="1.1" xmlns="http://www.w3.org/2000/svg"><defs><symbol id="topleft" viewBox="0 0 214 29" ><path d="M14.3 0.1L214 0.1 214 29 0 29C0 29 12.2 2.6 13.2 1.1 14.3-0.4 14.3 0.1 14.3 0.1Z"/></symbol><symbol id="topright" viewBox="0 0 214 29"><use xlink:href="#topleft"/></symbol><clipPath id="crop"><rect class="mask" width="100%" height="100%" x="0"/></clipPath></defs><svg width="50%" height="100%" transfrom="scale(-1, 1)"><use xlink:href="#topleft" width="214" height="29" class="chrome-tab-background"/><use xlink:href="#topleft" width="214" height="29" class="chrome-tab-shadow"/></svg><g transform="scale(-1, 1)"><svg width="50%" height="100%" x="-100%" y="0"><use xlink:href="#topright" width="214" height="29" class="chrome-tab-background"/><use xlink:href="#topright" width="214" height="29" class="chrome-tab-shadow"/></svg></g></svg>
        </div>
        <div class="chrome-tab-favicon" :style="{ backgroundImage: 'url(' + tab.favicon + ')' }"></div>
        <div class="chrome-tab-title">{{ tab.title }}</div>
        <div class="chrome-tab-close" @click.stop="removeTabAt(tabIndex)"></div>
      </div>
    </div>
    <div
      ref="bottom"
      class="chrome-tabs-bottom-bar"
      :style="{ zIndex: tabs.length + 1 }"></div>
  </div>
</template>

<script>
import Draggabilly from 'draggabilly'
const defaultTabProperties = {
  title: '',
  favicon: '',
}
let instanceId = 0

export default {
  name: 'chrome-tabs',

  props: {
    tabOverlapDistance: {
      type: Number,
      default: 14,
    },
    tabWidth: {
      type: Number,
      default: 243,
    },
    minWidth: {
      type: Number,
      default: 45,
    },
    maxWidth: {
      type: Number,
      default: 243,
    },
  },

  data () {
    return {
      tabs: [],
      currentTab: null,
      instanceId: instanceId++,
      tabId: 0,
    }
  },

  computed: {
    tabWidth () {
      const tabsContentWidth = this.$refs.content.clientWidth - this.tabOverlapDistance
      const width = (tabsContentWidth / this.tabs.length) + this.tabOverlapDistance
      return Math.max(this.minWidth, Math.min(this.maxWidth, width))
    },
    tabEffectiveWidth () {
      return this.tabWidth - this.tabOverlapDistance
    },
    tabPositions () {
      const tabEffectiveWidth = this.tabEffectiveWidth
      let left = 0
      let positions = []

      this.tabs.forEach((tabEl, i) => {
        positions.push(left)
        left += tabEffectiveWidth
      })
      return positions
    },
  },

  methods: {
    layoutTabs () {
      const tabWidth = this.tabWidth

      this.cleanUpPreviouslyDraggedTabs()

      requestAnimationFrame(() => {
        let styleHTML = ''
        this.tabPositions.forEach((left, i) => {
          styleHTML += `
            .chrome-tabs[data-chrome-tabs-instance-id="${this.instanceId}"] .chrome-tab:nth-child(${i + 1}) {
              transform: translate3d(${left}px, 0, 0)
            }
          `
        })
        this.animationStyleEl.innerHTML = styleHTML
      })
    },
    cleanUpPreviouslyDraggedTabs () {
      if (this.$refs.tabs) {
        this.$refs.tabs.forEach((tabEl) => tabEl.classList.remove('chrome-tab-just-dragged'))
      }
    },
    addTab (tabProperties, inBackground) {
      tabProperties = Object.assign({}, defaultTabProperties, tabProperties)
      if ('id' in tabProperties) {
        if (+tabProperties.id === tabProperties.id) {
          this.tabId = Math.max(this.tabId, tabProperties.id) + 1
        }
      } else {
        tabProperties.id = this.tabId++
      }

      const tabIndex = this.tabs.push(tabProperties) - 1

      if (!inBackground) {
        this.currentTab = tabProperties.id
      }

      this.$nextTick(() => {
        const el = this.$refs.tabs[tabIndex]
        el.classList.add('chrome-tab-just-added')
        setTimeout(() => el.classList.remove('chrome-tab-just-added'), 500)

        this.layoutTabs()
        this.setupDraggabilly()
      })
    },
    removeTab (tab) {
      for (let i = 0; i < this.tabs.length; i++) {
        if (this.tabs[i] === tab || this.tabs[i].id === tab) {
          this.removeTabAt(i);
          return this.tabs[i];
        }
      }
      return null;
    },
    removeTabAt (tabIndex) {
      if (this.currentTab === this.tabs[tabIndex].id) {
        if (tabIndex > 0) {
          this.currentTab = this.tabs[tabIndex - 1].id
        } else if (tabIndex < this.tabs.length - 1) {
          this.currentTab = this.tabs[tabIndex + 1].id
        } else {
          this.currentTab = null
        }
      }

      this.tabs.splice(tabIndex, 1)

      this.$nextTick(() => {
        this.layoutTabs()
        this.setupDraggabilly()
      })
    },
    setupDraggabilly () {
      const tabEls = this.$refs.tabs || []
      const tabEffectiveWidth = this.tabEffectiveWidth
      const tabPositions = this.tabPositions

      this.draggabillyInstances.forEach(draggabillyInstance => draggabillyInstance.destroy())

      tabEls.forEach((tabEl, originalIndex) => {
        const originalTabPositionX = tabPositions[originalIndex]
        const draggabillyInstance = new Draggabilly(tabEl, {
          axis: 'x',
          containment: this.$refs.content,
        })

        this.draggabillyInstances.push(draggabillyInstance)

        draggabillyInstance.on('dragStart', () => {
          this.cleanUpPreviouslyDraggedTabs()
          tabEl.classList.add('chrome-tab-currently-dragged')
          this.$el.classList.add('chrome-tabs-sorting')
        })

        draggabillyInstance.on('dragEnd', () => {
          const finalTranslateX = parseFloat(tabEl.style.left, 10)
          tabEl.style.transform = `translate3d(0, 0, 0)`

          // Animate dragged tab back into its place
          requestAnimationFrame(() => {
            tabEl.style.left = '0'
            tabEl.style.transform = `translate3d(${finalTranslateX}px, 0, 0)`

            requestAnimationFrame(() => {
              tabEl.classList.remove('chrome-tab-currently-dragged')
              this.$el.classList.remove('chrome-tabs-sorting')

              tabEl.classList.add('chrome-tab-just-dragged')

              requestAnimationFrame(() => {
                tabEl.style.transform = ''

                this.setupDraggabilly()
              })
            })
          })
        })

        draggabillyInstance.on('dragMove', (event, pointer, moveVector) => {
          // Current index be computed within the event since it can change during the dragMove
          const tabEls = this.$refs.tabs || []
          const currentIndex = tabEls.indexOf(tabEl)

          const currentTabPositionX = originalTabPositionX + moveVector.x
          const destinationIndex = Math.max(0, Math.min(tabEls.length, Math.floor((currentTabPositionX + (tabEffectiveWidth / 2)) / tabEffectiveWidth)))

          if (destinationIndex < currentIndex) {
            tabEl.parentNode.insertBefore(tabEl, this.$refs.tabs[destinationIndex])
          } else
          if (destinationIndex > currentIndex) {
            tabEl.parentNode.insertBefore(tabEl, this.$refs.tabs[destinationIndex + 1])
          }
        })
      })
    },
  },

  created () {
    this.draggabillyInstances = []
  },

  mounted () {
    this.animationStyleEl = document.createElement('style')
    this.$el.appendChild(this.animationStyleEl)

    window.addEventListener('resize', event => this.layoutTabs())

    this.layoutTabs()
    this.setupDraggabilly()
  },
}
</script>

<style scoped>
.chrome-tabs {
  box-sizing: border-box;
  position: relative;
  font-size: 10px;
  height: 4.2em;
  background: linear-gradient(#dad9da, #d9d8d9);
  padding: 1em 1.2em 0.45em 1.2em;
  border-radius: 0.5em 0.5em 0 0;
  box-shadow: 0 0.05em #b7b7b7;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
}
.chrome-tabs * {
  box-sizing: inherit;
  font-family: inherit;
}
.chrome-tabs .chrome-tabs-bottom-bar {
  position: absolute;
  bottom: 0;
  height: 0.45em;
  left: 0;
  width: 100%;
  background: #f2f2f2;
  box-shadow: 0 -0.05em rgba(0,0,0,0.27);
  z-index: 20;
}
.chrome-tabs .chrome-tabs-content {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
.chrome-tabs .chrome-tab {
  position: absolute;
  left: 0;
  height: 2.8em;
  width: 24em;
  border: 0;
  margin: 0;
  z-index: 1;
}
.chrome-tabs .chrome-tab,
.chrome-tabs .chrome-tab * {
  user-select: none;
  cursor: default;
}
.chrome-tabs .chrome-tab .chrome-tab-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  pointer-events: none;
}
.chrome-tabs .chrome-tab .chrome-tab-background > svg {
  width: 100%;
  height: 100%;
}
.chrome-tabs .chrome-tab .chrome-tab-background > svg .chrome-tab-shadow {
  fill: none;
  stroke: rgba(0,0,0,0.27);
  stroke-width: 0.5px;
}
.chrome-tabs .chrome-tab .chrome-tab-background > svg .chrome-tab-background {
  fill: #d0d0d0;
  transform: translateX(0.25px) translateY(0.25px);
}
.chrome-tabs .chrome-tab.chrome-tab-current {
  z-index: 999;
}
.chrome-tabs .chrome-tab.chrome-tab-current .chrome-tab-background > svg .chrome-tab-background {
  fill: #f2f2f2;
}
.chrome-tabs .chrome-tab.chrome-tab-just-added {
  top: 10px;
  animation: chrome-tab-just-added 120ms forwards ease-in-out;
}
@-moz-keyframes chrome-tab-just-added {
  to {
    top: 0;
  }
}
@-webkit-keyframes chrome-tab-just-added {
  to {
    top: 0;
  }
}
@-o-keyframes chrome-tab-just-added {
  to {
    top: 0;
  }
}
@keyframes chrome-tab-just-added {
  to {
    top: 0;
  }
}
.chrome-tabs.chrome-tabs-sorting .chrome-tab:not(.chrome-tab-currently-dragged),
.chrome-tabs:not(.chrome-tabs-sorting) .chrome-tab.chrome-tab-just-dragged {
  transition: transform 120ms ease-in-out;
}
.chrome-tabs .chrome-tab-favicon {
  position: relative;
  margin-left: 1.6em;
  height: 1.6em;
  width: 1.6em;
  background-size: 1.6em;
  margin-top: 0.6em;
  z-index: 3;
  display: inline-block;
  vertical-align: top;
  pointer-events: none;
}
.chrome-tabs .chrome-tab-title {
  position: relative;
  display: inline-block;
  vertical-align: top;
  color: #222;
  padding: 0 0.25em;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  font-size: 1.2em;
  margin-top: 0.5em;
  max-width: calc(100% - 5em);
  pointer-events: none;
}
.chrome-tabs .chrome-tab-close {
  position: absolute;
  width: 1.4em;
  height: 1.4em;
  border-radius: 50%;
  z-index: 2;
  right: 1.4em;
  top: 0.7em;
}
.chrome-tabs .chrome-tab-close::before {
  content: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 14 14'><path stroke='#5a5a5a' stroke-linecap='round' stroke-width='1.125' d='M4 4 L10 10 M10 4 L4 10'></path></svg>");
  position: absolute;
  display: block;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}
.chrome-tabs .chrome-tab-close:hover::before,
.chrome-tabs .chrome-tab-close:hover:active::before {
  content: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 14 14'><path stroke='#fff' stroke-linecap='round' stroke-width='1.2' d='M4 4 L10 10 M10 4 L4 10'></path></svg>");
}
.chrome-tabs .chrome-tab-close:hover {
  background: #e25c4b;
}
.chrome-tabs .chrome-tab-close:hover:active {
  background: #b74a3b;
}

.chrome-tabs.chrome-tabs-dark-theme {
  background: linear-gradient(#1b1d1f, #1d1f21);
  box-shadow: 0 0.05em #282828;
}
.chrome-tabs.chrome-tabs-dark-theme .chrome-tabs-bottom-bar {
  background: #505050;
  box-shadow: 0 -0.05em rgba(0,0,0,0.5);
}
.chrome-tabs.chrome-tabs-dark-theme .chrome-tab .chrome-tab-background > svg .chrome-tab-shadow {
  stroke: rgba(0,0,0,0.5);
}
.chrome-tabs.chrome-tabs-dark-theme .chrome-tab .chrome-tab-background > svg .chrome-tab-background {
  fill: #373737;
}
.chrome-tabs.chrome-tabs-dark-theme .chrome-tab.chrome-tab-current .chrome-tab-background > svg .chrome-tab-background {
  fill: #505050;
}
.chrome-tabs.chrome-tabs-dark-theme .chrome-tab .chrome-tab-title {
  -webkit-font-smoothing: antialiased;
  color: #fff;
}
.chrome-tabs.chrome-tabs-dark-theme .chrome-tab .chrome-tab-close:not(:hover)::before {
  content: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 14 14'><path stroke='#cfcfcf' stroke-linecap='round' stroke-width='1.125' d='M4 4 L10 10 M10 4 L4 10'></path></svg>");
}

</style>
