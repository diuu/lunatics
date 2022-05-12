<script lang="ts">
import { computed, onBeforeMount, reactive, ref } from 'vue'

function fetchFcd (url: string) {
  return fetch(url).then(res => res.text())
}

function fetchJson(url: string) {
  return fetch(url).then(res => res.json())
}

export default {
  setup() {
    const luna = reactive({
      totalSupply: 0,
      circulatingSupply: 0,
      price: 0,
      marketCap: computed(() => luna.price * luna.circulatingSupply),
    })

    const ust = reactive({
      totalSupply: 0,
      circulatingSupply: 0,
      price: 0,
      marketCap: computed(() => ust.price * ust.circulatingSupply),
    })

    const updatedAt = ref(new Date())

    function fetchData () {
      fetchFcd('https://fcd.terra.dev/v1/TotalSupply/luna').then(body => luna.totalSupply = Number(body))
      fetchFcd('https://fcd.terra.dev/v1/circulatingsupply/luna').then(body => luna.circulatingSupply = Number(body))
      fetchJson('https://api.coingecko.com/api/v3/simple/price?ids=terra-luna&vs_currencies=usd').then((res) => luna.price = res[ 'terra-luna' ][ 'usd' ])

      fetchFcd('https://fcd.terra.dev/v1/TotalSupply/ust').then(body => ust.totalSupply = Number(body))
      fetchFcd('https://fcd.terra.dev/v1/circulatingsupply/ust').then(body => ust.circulatingSupply = Number(body))
      fetchJson('https://api.coingecko.com/api/v3/simple/price?ids=terrausd&vs_currencies=usd').then((res) => ust.price = res[ 'terrausd' ][ 'usd' ])

      updatedAt.value = new Date()
    }

    onBeforeMount(fetchData)
    setInterval(fetchData, 30 * 1000)

    return {
      luna,
      ust,
      updatedAt,
      onClickRefresh: fetchData
    }
  }
}
</script>

<template>
  <h1 style="margin: 0">Lunatics</h1>
  <p style="margin: 0">
    <small>
      Updated At {{ updatedAt.toLocaleTimeString()}}
      <button @click="onClickRefresh">Refresh</button>
    </small>
  </p>

  <div class="page-content">
    <div class="content_token _luna">
      <h2>LUNA</h2>

      <h4>Total Supply</h4>
      {{luna.totalSupply.toLocaleString()}}

      <h4>Circulating Supply</h4>
      {{luna.circulatingSupply.toLocaleString()}}

      <h4>Price</h4>
      {{ luna.price }}

      <h4>Market Cap</h4>
      {{ luna.marketCap.toLocaleString() }}
    </div>

    <div class="content_token _ust">
      <h2>UST</h2>

      <h4>Total Supply</h4>
      {{ust.totalSupply.toLocaleString()}}

      <h4>Circulating Supply</h4>
      {{ust.circulatingSupply.toLocaleString()}}


      <h4>Price</h4>
      {{ ust.price }}

      <h4>Market Cap</h4>
      {{ ust.marketCap.toLocaleString() }}
    </div>
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

  .page-content {
    margin-top: 20px;
    display: flex;
    justify-content: space-between;
    align-self: stretch;

    .content_token {
      flex: 1;
      flex-direction: column;
      text-align: center;

      h4 {
        margin-top: 50px;
      }
    }
  }

  .fork-on-github {
    position: fixed;
    top: 0;
    right: 0;
  }
}
</style>
