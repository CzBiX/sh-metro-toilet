<template>
<header>
  <h1>上海地铁卫生间</h1>
</header>

<main>
  <ol class="lines">
    <li v-for="(_, key) of lines"
      :key="key"
    >
      <button :class="getLineClasses(key)"
      @click="selectLineKey = key"
      >
        {{ getLineName(key) }}
      </button>
    </li>
  </ol>
  <div class="stations" :class="'line-' + selectLineKey" v-if="selectLineKey">
    <ol class="stations-header">
      <li v-for="key of selectLine"
        :key="key"
      >
        <a :href="'#' + getStationId(key)">
          <span>{{ getStationName(key) }}</span>
          <i v-if="hasToilets(key)" class="toilet-icon" :class="getToiletIconForTitle(key)" />
        </a>
      </li>
    </ol>
    <ol class="station-list" ref="stationList">
      <li v-for="key of selectLine" :key="key" :id="getStationId(key)">
        <p class="station-name">{{ getStationName(key) }}</p>
        <p class="toilet-section" v-for="(toilet, index) of getToilets(key)" :key="index">
          <span class="line-name">{{ toilet.line }}</span>
          <i class="toilet-icon" :class="getToiletIcon(toilet)" />
          <span class="category">{{ toilet.category }}</span><br/>
          <span class="place">{{ toilet.place }}</span>
        </p>
      </li>
    </ol>
  </div>
</main>

<footer>
  <p>数据来源于<a ref="no-referrer" href="http://service.shmetro.com/i/cw.html">上海地铁</a></p>
  <p>最后更新时间：{{ lastModifiedStr }}</p>
  <p>信息仅供参考，请以实际为准</p>
</footer>
</template>

<script lang="ts">
import { computed, defineComponent, ref } from 'vue'
import { lines, stations } from './station-data'

import { lastModified, toilets } from './toilet-data.json'

type LineKey = keyof typeof lines
type ToiletKey = keyof typeof toilets

interface ToiletInfo {
	line: string
	category: string
	place: string
}

function isOutside(toilet: ToiletInfo) {
  return toilet.category !== '付费区'
}

function useLines() {
  const selectLineKey = ref<LineKey>()

  const selectLine = computed(() => {
    const v = selectLineKey.value!

    return lines[v]
  })

  function getLineName(key: string) {
    if (key === '41') {
      return '浦江线'
    }

    return key + '号线'
  }

  function getLineClasses(key: string) {
    const classes = [
      'line-' + key,
    ]

    if (key === selectLineKey.value) {
      classes.push('selected')
    }

    return classes
  }

  return {
    selectLineKey,
    selectLine,
    getLineName,
    getLineClasses,
  }
}

function useStations() {
  const stationList = ref<HTMLElement>()

  function getStationName(key: string) {
    return stations[key]
  }

  function getStationId(key: string) {
    return 'station-' + key
  }

  return {
    stationList,
    getStationId,
    getStationName,
  }
}

function useToilets() {
  function getToilets(key: string) {
    return toilets[key as ToiletKey] as ToiletInfo[]
  }

  function hasToilets(key: string) {
    return getToilets(key)?.length > 0
  }

  function getToiletIconClass(isOutside: boolean) {
    return isOutside ? 'outside' : 'inside'
  }

  function getToiletIcon(toilet: ToiletInfo) {
    return getToiletIconClass(isOutside(toilet))
  }

  function getToiletIconForTitle(key: string) {
    const toilets = getToilets(key)
    if (!toilets || toilets.length === 0) {
      return null
    }

    for (const toilet of toilets) {
      if (!isOutside(toilet)) {
        return getToiletIconClass(true)
      }
    }

    return getToiletIconClass(false)
  }

  const lastModifiedStr = new Date(lastModified).toLocaleString()

  return {
    hasToilets,
    getToilets,
    getToiletIconForTitle,
    getToiletIcon,
    lastModifiedStr,
  }
}

export default defineComponent({
  name: 'App',
  setup() {
    return {
      lines,
      stations,
      ...useLines(),
      ...useStations(),
      ...useToilets(),
    }
  }
})
</script>

<style lang="scss">
@import 'assets/lines-color.scss';

body {
  overflow-y: scroll;
}

footer {
  margin-top: 1em;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: auto;
  padding: 0 2em;
  max-width: 1024px;
}

%li-button {
  border: none;
  outline: none;
  background: none;
}

.lines {
  padding-inline-start: 0;
  list-style: none;

  display: grid;
  grid-template-columns: repeat(auto-fit, 4em);
  gap: 0.4em;

  button {
    @extend %li-button;

    width: 100%;
    padding: 0.4em 0 0.2em;
    border-bottom: 0.2em solid var(--fg-color);
    cursor: pointer;

    &:hover {
      background-color: #eee;
    }

    &.selected {
      padding: 0.4em 0;
      border: none;
      background-color: var(--fg-color);
      color: #fff;
    }
  }
}

.stations-header {
  padding-inline-start: 0;
  list-style: none;

  display: flex;
  flex-wrap: wrap;
  padding: 0.2em;
  border: 2px dashed var(--fg-color);

  a {
    display: block;
    color: black;
    padding: 0.4em;
    text-decoration: none;

    &:hover {
      background-color: #eee;
    }
  }
}

.station-list {
  text-align: left;
  border-bottom: 2px dashed var(--fg-color);
  padding: 0.4em 0;

  .station-name:not(:last-child) {
    margin-bottom: 0.4em;
  }

  .toilet-section {
    margin: 0.4em 0;
  }

  .line-name {
    display: inline-block;
    width: 4em;
  }

  .category {
    display: inline-block;
    width: 4em;
    margin-right: 1em;
  }

  p {
    margin-block: 0;
  }

  li {
    padding: 0.4em;
  }

  li:nth-child(2n) {
    background-color: #eee;
  }
}

.toilet-icon {
  vertical-align: top;
  display: inline-block;
  width: 22px;
  height: 22px;

  &.inside {
    background: url('./assets/toilet.png');
  }

  &.outside {
    background: url('./assets/toilet-outside.png');
  }
}
</style>