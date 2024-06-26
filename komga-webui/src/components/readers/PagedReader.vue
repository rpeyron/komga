<template>
  <div
    v-touch="{
               left: () => {if(swipe) {turnRight()}},
               right: () => {if(swipe) {turnLeft()}},
               up: () => {if(swipe) {verticalNext()}},
               down: () => {if(swipe) {verticalPrev()}}
             }"
  >

    <!--  clickable zones (not active with pan-ozom) 
    <div v-if="!vertical"  @click="turnLeft()" class="left-quarter" />
    <div v-if="!vertical"  @click="turnRight()" class="right-quarter" />
    <div v-if="vertical"   @click="verticalPrev()" class="top-quarter" />
    <div v-if="vertical"   @click="verticalNext()" class="bottom-quarter" />
    -->

    <!--  clickable zone: menu  
    <div @click="centerClick()" :class="`${vertical ? 'center-vertical' : 'center-horizontal'}`"  />
    -->
  
  <v-zoomer style="width:100%"
            :aspect-ratio="contentAspectRatio"             
            >
      <div v-on:mousemove.prevent v-on:mouseup="handleMouseUp()" v-on:mousedown="handleMouseDown()"
           v-on:touchstart.prevent v-on:touchend.prevent v-on:touchmove.prevent
           v-on:click="handleClick()" v-on:dblclick="handleDoubleClick()">
      <v-carousel v-model="carouselPage"
                :show-arrows="false"
                :continuous="false"
                :reverse="flipDirection"
                :vertical="vertical"
                hide-delimiters
                touchless
                height="100%" 
    >
      <v-carousel-item v-for="(spread, i) in spreads"
                       :key="`spread${i}`"
                       :eager="eagerLoad(i)"
                       class="full-height"
                       :class="preRender(i) ? 'pre-render' : ''"
                       :transition="animations ? undefined : false"
                       :reverse-transition="animations ? undefined : false"
      >
        <div class="full-height d-flex flex-column justify-center">
          <div :class="`d-flex flex-row${flipDirection ? '-reverse' : ''} justify-center px-0 mx-0`">
          <div v-if="!vertical" @click.stop="turnLeft()" style="height: 100%; flex: 1 0 0;"></div>
          <div v-if="vertical" @click.stop="verticalPrev()" style="width: 100%; flex: 1 0 0;"></div>
              <span ref="`images${i}`">
            <img v-for="(page, j) in spread"
                 :alt="`Page ${page.number}`"
                 :key="`spread${i}-${j}`"
                 :src="page.url"
                 :class="imgClass(spread)"
                 class="img-fit-all"
                 @load="calcImageRatio"
            />
          </span>
          <div v-if="!vertical" @click.stop="turnRight()" style="height: 100%; flex: 1 0 0;"></div>
          <div v-if="vertical" @click.stop="verticalNext()" style="width: 100%; flex: 1 0 0;"></div>
          </div>
        </div>
      </v-carousel-item>
    </v-carousel> 
  </div>
  </v-zoomer>

  <div v-if="!vertical" @click="turnLeft()" class="page-button page-left">&#x1F780;&nbsp;</div>
  <div v-if="!vertical" @click="turnRight()" class="page-button page-right">&nbsp;&#x1F782;</div>
  <div v-if="vertical" @click="verticalPrev()" class="page-button page-top">&#x1F781;</div>
  <div v-if="vertical" @click="verticalNext()" class="page-button page-bottom">&#x1F783;</div>

</div>
</template>

<script lang="ts">
import Vue from 'vue'
import {ReadingDirection} from '@/types/enum-books'
import {PagedReaderLayout, ScaleType} from '@/types/enum-reader'
import {shortcutsLTR, shortcutsRTL, shortcutsVertical} from '@/functions/shortcuts/paged-reader'
import {PageDtoWithUrl} from '@/types/komga-books'
import {buildSpreads} from '@/functions/book-spreads'
import {debounce} from 'lodash'

export default Vue.extend({
  name: 'PagedReader',
  data: function () {
    return {
      logger: 'PagedReader',
      carouselPage: 0,
      spreads: [] as PageDtoWithUrl[][],
      contentAspectRatio: 1.0,
      singleClickTimeout: undefined as any,
      singleClickCanClick: true,
      noDragCanClick: true,
    }
  },
  props: {
    pages: {
      type: Array as () => PageDtoWithUrl[],
      required: true,
    },
    page: {
      type: Number,
      required: true,
    },
    pageLayout: {
      type: String as () => PagedReaderLayout,
      required: true,
    },
    animations: {
      type: Boolean,
      required: true,
    },
    swipe: {
      type: Boolean,
      required: true,
    },
    readingDirection: {
      type: String as () => ReadingDirection,
      required: true,
    },
    scale: {
      type: String as () => ScaleType,
      required: true,
    },
  },
  watch: {
    pages: {
      handler(val) {
        this.spreads = buildSpreads(val, this.pageLayout)
      },
      immediate: true,
    },
    carouselPage(val, old) {
      this.$debug('[watch:carouselPage', `old:${old}`, `new:${val}`)
      if (this.carouselPage >= 0 && this.carouselPage < this.spreads.length && this.spreads.length > 0) {
        const currentSpread = this.spreads[this.carouselPage]
        const currentPage = currentSpread.length == 2 && currentSpread[1].mediaType ? currentSpread[1] : currentSpread[0]
        this.$emit('update:page', currentPage.number)
      } else {
        this.$emit('update:page', 1)
      }
      this.calcImageRatio()
    },
    page(val, old) {
      this.$debug('[watch:page]', `old:${old}`, `new:${val}`)
      const spreadIndex = this.toSpreadIndex(val)
      this.$debug('[watch:page]', `toSpreadIndex:${spreadIndex}`)
      this.carouselPage = spreadIndex
    },
    pageLayout: {
      handler(val) {
        const current = this.page
        this.spreads = buildSpreads(this.pages, val)
        this.carouselPage = this.toSpreadIndex(current)
      },
      immediate: true,
    },
  },
  created() {
    window.addEventListener('keydown', this.keyPressed)
  },
  destroyed() {
    window.removeEventListener('keydown', this.keyPressed)
  },
  computed: {
    shortcuts(): any {
      const shortcuts = []
      switch (this.readingDirection) {
        case ReadingDirection.LEFT_TO_RIGHT:
          shortcuts.push(...shortcutsLTR)
          break
        case ReadingDirection.RIGHT_TO_LEFT:
          shortcuts.push(...shortcutsRTL)
          break
        case ReadingDirection.VERTICAL:
          shortcuts.push(...shortcutsVertical)
          break
      }
      return this.$_.keyBy(shortcuts, x => x.key)
    },
    flipDirection(): boolean {
      return this.readingDirection === ReadingDirection.RIGHT_TO_LEFT
    },
    vertical(): boolean {
      return this.readingDirection === ReadingDirection.VERTICAL
    },
    currentSlide(): number {
      return this.carouselPage + 1
    },
    slidesCount(): number {
      return this.spreads.length
    },
    canPrev(): boolean {
      return this.currentSlide > 1
    },
    canNext(): boolean {
      return this.currentSlide < this.slidesCount
    },
    isDoublePages(): boolean {
      return this.pageLayout === PagedReaderLayout.DOUBLE_PAGES || this.pageLayout === PagedReaderLayout.DOUBLE_NO_COVER
    },
  },
  methods: {
    keyPressed(e: KeyboardEvent) {
      this.shortcuts[e.key]?.execute(this)
    },
    imgClass(spread: PageDtoWithUrl[]): string {
      const double = spread.length > 1
      switch (this.scale) {
        case ScaleType.WIDTH:
          return double ? 'img-double-fit-width' : 'img-fit-width'
        case ScaleType.WIDTH_SHRINK_ONLY:
          return double ? 'img-double-fit-width-shrink-only' : 'img-fit-width-shrink-only'
        case ScaleType.HEIGHT:
          return 'img-fit-height'
        case ScaleType.SCREEN:
          return double ? 'img-double-fit-screen' : 'img-fit-screen'
        default:
          return 'img-fit-original'
      }
    },
    eagerLoad(spreadIndex: number): boolean {
      return Math.abs(this.carouselPage - spreadIndex) <= 2
    },
    preRender(spreadIndex: number): boolean {
      return Math.abs(this.carouselPage - spreadIndex) > (this.animations ? 1 : 0)
    },
    centerClick() {
      this.$emit('menu')
    },
    handleMouseDown() {
      document.addEventListener('mousemove', this.handleMouseMove)
    },
    handleMouseMove() {
      this.noDragCanClick = false
    },
    handleMouseUp() {
      document.removeEventListener('mousemove', this.handleMouseMove)
      setTimeout(() => this.noDragCanClick = true, 50)
    },
    handleClick() {
      if (this.singleClickCanClick && this.noDragCanClick) {
        this.singleClickTimeout = setTimeout(() => this.centerClick(), 300) 
        this.singleClickCanClick = false
        setTimeout(() => this.singleClickCanClick = true, 500)
      }
    },
    handleDoubleClick() {
      if (this.singleClickTimeout) clearTimeout(this.singleClickTimeout)
    },
    turnRight() {
      if (!this.vertical)
        this.flipDirection ? this.prev() : this.next()
    },
    turnLeft() {
      if (!this.vertical)
        this.flipDirection ? this.next() : this.prev()
    },
    verticalPrev() {
      if (this.vertical) this.prev()
    },
    verticalNext() {
      if (this.vertical) this.next()
    },
    prev() {
      if (this.canPrev) {
        this.carouselPage--
        window.scrollTo(0, 0)
      } else {
        this.$emit('jump-previous')
      }
    },
    next() {
      if (this.canNext) {
        this.carouselPage++
        window.scrollTo(0, 0)
      } else {
        this.$emit('jump-next')
      }
    },
    toSpreadIndex(i: number): number {
      this.$debug('[toSpreadIndex]', `i:${i}`, `isDoublePages:${this.isDoublePages}`)
      if (this.spreads.length > 0) {
        if (this.isDoublePages) {
          for (let j = 0; j < this.spreads.length; j++) {
            for (let k = 0; k < this.spreads[j].length; k++) {
              if (this.spreads[j][k].number === i) {
                return j
              }
            }
          }
        } else {
          return i - 1
        }
      }
      return i - 1
    },
    calcImageRatio(payload: Event) {
      let imagesSpan = (payload.target)?payload.target.parentElement:null
      //let curImagesSpan = this.$refs[`images${this.carouselPage}`]
      if (imagesSpan) {
        this.contentAspectRatio = imagesSpan.clientWidth / imagesSpan.clientHeight
      }
    },
  },
})
</script>
<style scoped>
.full-height {
  height: 100%;
}

.page-button {
  opacity: 50%;
  background-color: lightgrey;
  border-radius: 50%;
  height: 6%;
  aspect-ratio: 1;
  opacity: 50%;
  justify-content: center;
  align-items: center;
  display: flex;
  font-size: xx-large;
  color: darkgrey;
}

.page-button:hover {
  opacity: 90%;
}

.page-left { position: absolute; top: 50%; left: 10px; transform: translate(0%, -50%); }
.page-right { position: absolute; top: 50%; right: 10px; transform: translate(0%, -50%); }
.page-top { position: absolute; left: 50%; top: 10px; transform: translate(-50%, 0%); }
.page-bottom { position: absolute; left: 50%; bottom: 10px; transform: translate(-50%, 0%); }

.left-quarter {
  top: 0;
  left: 0;
  width: 25%;
  height: 100%;
  position: absolute;
}

.right-quarter {
  top: 0;
  right: 0;
  width: 25%;
  height: 100%;
  position: absolute;
}

.top-quarter {
  top: 0;
  height: 25%;
  width: 100%;
  position: absolute;
}

.bottom-quarter {
  bottom: 0;
  height: 25%;
  width: 100%;
  position: absolute;
}

.center-horizontal {
  top: 0;
  left: 25%;
  width: 50%;
  height: 100%;
  position: absolute;
}

.center-vertical {
  top: 25%;
  height: 50%;
  width: 100%;
  position: absolute;
}

.img-fit-all {
  object-fit: contain;
  object-position: center;
}

.img-fit-width {
  width: 100vw;
  min-height: 100vh;
  align-self: flex-start;
}

.img-double-fit-width {
  width: 50vw;
  min-height: 100vh;
  align-self: flex-start;
}

.img-fit-width-shrink-only {
  max-width: 100vw;
  align-self: flex-start;
}

.img-double-fit-width-shrink-only {
  max-width: 50vw;
  align-self: flex-start;
}

.img-fit-original {
  width: auto;
  height: auto;
}

.img-fit-height {
  min-height: 100vh;
  height: 100vh;
}

.img-fit-screen {
  max-width: 100vw;
  max-height: 100vh;
}

.img-double-fit-screen {
  max-width: 50vw;
  height: 100vh;
}

.pre-render {
  display: block !important;
  position: fixed;
  right: -1000vw;
  top: -1000vh;
}
</style>
