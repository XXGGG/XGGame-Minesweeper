<script setup lang="ts">
import { isDev, toggleDev } from '~/composables'
import { GamePlay } from '~/composables/logic'

const play = new GamePlay(12, 12)
useStorage('XGGameMinesweeperState',play.state)
const state = computed(() => play.board)

</script>

<template>
  <div>
    XGGame - Minesweeper

    <div p5>
      <div v-for="row, y in state" :key="y" flex="~" items-center justify-center>
        <MineBlock v-for="block, x in row" :key="x" :block="block" @click="play.onClick(block)"
          @contextmenu.prevent="play.onRightClick(block)">
        </MineBlock>
      </div>
    </div>
  </div>

  <div flex='~ gap-1' justify-center>
    <button btn @click="toggleDev()">
      {{ isDev ? '作弊模式' : '正常模式' }}
    </button>
    <button btn @click="play.reset()">
      重新开始
    </button>
  </div>
</template>

