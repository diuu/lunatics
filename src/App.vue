<script lang="ts">
import { useStorage } from '@vueuse/core'
import { computed, onBeforeMount, Ref, unref } from 'vue'

function fetchFcd (url: string) {
  return fetch(url).then(res => res.text()).then(text => Number(text))
}

function fetchJson(url: string) {
  return fetch(url).then(res => res.json())
}

export default {
  setup() {
    const luna = useStorage('luna', {
      totalSupply: 29101617313.334843,
      circulatingSupply: 0,
      price: 0,
      updatedAt: 1652366249245,
      lastTime: {
        totalSupply: 29101617313.334843,
        updatedAt: 1652366249245,
      },
    })

    const lunaRef = {
      marketCap: computed(() => luna.value.price * luna.value.circulatingSupply),
      issueRate: computed(() => {
        return luna.value.totalSupply === 0 ? 0 : (luna.value.totalSupply - luna.value.lastTime.totalSupply) / ((Date.now() - luna.value.lastTime.updatedAt) / 1000) * 3600
      })
    }

    const ust = useStorage('ust', {
      totalSupply: 11852549719.719362,
      circulatingSupply: 0,
      price: 0,
      updatedAt: 1652366249245,
      lastTime: {
        totalSupply: 11852549719.719362,
        updatedAt: 1652366249245,
      },
    })

    const ustRef = {
      marketCap: computed(() => ust.value.price * ust.value.circulatingSupply),
      issueRate: computed(() => {
        return ust.value.totalSupply === 0 ? 0 : (ust.value.lastTime.totalSupply - ust.value.totalSupply) / ((Date.now() - ust.value.lastTime.updatedAt) / 1000) * 3600
      })
    }

    const simple = useStorage('simple', false)

    function onClickSwitch() {
      simple.value = !simple.value
    }

    function displayLargeNumber(number: number|string|Ref<number>) {
      const value = unref(number)
      return simple.value ? (Number(value) / 10**9).toFixed(2) + 'B' : Number(Number(value).toFixed(0)).toLocaleString()
    }
    function fetchData () {
      fetchFcd('https://fcd.terra.dev/v1/TotalSupply/luna').then(body => {
        if (luna.value.totalSupply !== body) {
          luna.value.lastTime.totalSupply = luna.value.totalSupply
          luna.value.lastTime.updatedAt = luna.value.updatedAt
          luna.value.totalSupply = body
        }
        luna.value.updatedAt = Date.now()
      })
      fetchFcd('https://fcd.terra.dev/v1/circulatingsupply/luna').then(body => luna.value.circulatingSupply = body)
      fetchJson('https://api.coingecko.com/api/v3/simple/price?ids=terra-luna&vs_currencies=usd').then((res) => luna.value.price = res[ 'terra-luna' ][ 'usd' ])

      fetchFcd('https://fcd.terra.dev/v1/TotalSupply/ust').then(body => {
        if (ust.value.totalSupply !== body) {
          ust.value.lastTime.totalSupply = ust.value.totalSupply
          ust.value.lastTime.updatedAt = ust.value.updatedAt
          ust.value.totalSupply = body
        }
        ust.value.updatedAt = Date.now()
      })
      fetchFcd('https://fcd.terra.dev/v1/circulatingsupply/ust').then(body => ust.value.circulatingSupply = body)
      fetchJson('https://api.coingecko.com/api/v3/simple/price?ids=terrausd&vs_currencies=usd').then((res) => ust.value.price = res[ 'terrausd' ][ 'usd' ])
    }

    onBeforeMount(fetchData)
    setInterval(fetchData, 30 * 1000)

    return {
      luna,
      lunaRef,
      ust,
      ustRef,
      simple,
      displayLargeNumber,
      onClickRefresh: fetchData,
      onClickSwitch,
    }
  }
}
</script>

<template>
  <h1 style="margin: 0">Lunatics</h1>
  <p style="margin: 0">
    <small>
      Updated At {{ new Date(luna.updatedAt).toLocaleTimeString()}}
      <button @click="onClickRefresh">Refresh</button>
    </small>
  </p>

  <div class="page-content">
    <div class="content_token _luna">
      <h2>LUNA</h2>

      <h4>Total Supply</h4>
      {{displayLargeNumber(luna.totalSupply)}}

      <h4>Circulating Supply</h4>
      {{displayLargeNumber(luna.circulatingSupply)}}

      <h4>Issue Rate</h4>
      {{displayLargeNumber(lunaRef.issueRate)}} / Hour

      <h4>Price</h4>
      ${{ luna.price }}

      <h4>Market Cap</h4>
      ${{ displayLargeNumber(lunaRef.marketCap) }}
    </div>

    <div class="content_token _ust">
      <h2>UST</h2>

      <h4>Total Supply</h4>
      {{displayLargeNumber(ust.totalSupply)}}

      <h4>Circulating Supply</h4>
      {{displayLargeNumber(ust.circulatingSupply)}}

      <h4>Burn Rate</h4>

      {{displayLargeNumber(ustRef.issueRate)}} / Hour

      <h4>Price</h4>
      ${{ ust.price }}

      <h4>Market Cap</h4>
      ${{ displayLargeNumber(ustRef.marketCap) }}
    </div>
  </div>

  <div>
    <button @click="onClickSwitch">{{ simple ? 'Complete View' : 'Simple View' }}</button>
  </div>
  <a class="fork-on-github" href="https://github.com/zgayjjf/lunatics"><img loading="lazy" width="149" height="149" src="https://github.blog/wp-content/uploads/2008/12/forkme_right_white_ffffff.png?resize=149%2C149" class="attachment-full size-full" alt="Fork me on GitHub" data-recalc-dims="1"></a>
</template>

<style lang="scss">
html {
  height: 100%;
}
body {
  margin: 0;
  height: 100%;
}
#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  font-family: sans-serif;
  background-color: #173e9f;
  color: #fff;

  button {
    background-color: #6093f0;
    border: none;
    border-radius: 20px;
    color: #fff;
    padding: 4px 8px;
    cursor: pointer;

    &:hover {
      background-color: #5083e0;
    }

    &:active {
      background-color: #4063a0;
    }
  }

  h4 {
    margin-top: 30px;
    margin-bottom: 10px;
  }

  .page-content {
    margin-top: 20px;
    display: flex;
    justify-content: space-between;
    align-self: stretch;

    .content_token {
      flex: 1;
      flex-direction: column;
      text-align: center;
    }
  }

  .fork-on-github {
    position: fixed;
    top: 0;
    right: 0;
  }
}
</style>
