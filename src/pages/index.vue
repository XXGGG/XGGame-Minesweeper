<script setup lang="ts">
import { BlockState } from '~/types'

const WIDTH = 10
const HEIGHT = 10
//定义一个 10*10的二维数组
const state = ref(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from({ length: WIDTH }, (_, x): BlockState => ({
      x,
      y,
      adjacentMines: 0,
      revealed: false
    })))
)

//定义炸弹！【初始化，在计算炸弹的时候，在第一次点击的周围不要生成炸弹！】
function generateMines(initial: BlockState) { //initial 的【点击的坐标】
  for (const row of state.value) {
    for (const block of row) {
      //如果第一下点击传过来的坐标x - 现在遍历的坐标x 左右正负都小于1 也就是在周围就continue!
      if (Math.abs(initial.x - block.x) < 1) {
        continue //continue 就是跳过这个循环遍历 进行下一次遍历
      }
      if (Math.abs(initial.y - block.y) < 1) {//相同原理
        continue
      }
      block.mine = Math.random() < 0.2
    }
  }
  updateNumbers()
}

//计算附近有的炸弹 [directions/方向]
const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
]
//循环每一个【数】，然后把这个【数】加上上面的数值，就是它的 8个方位！
function updateNumbers() {
  state.value.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine)
        return
      //当它是个炸弹，就短路！

      //然后计算它的各个范围的炸弹数量👇
      getSiblings(block) //它会返回这个坐标周围的那个坐标是炸弹的【数组】
        .forEach(b => { //数组每一个元素就代表这个周围有多少个炸弹 就加1
          if (b.mine)
            block.adjacentMines += 1
        })
    })
  })
}

//重构 - 把方向筛选的方法提取出来
function getSiblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx;
    const y2 = block.y + dy;
    //如果x2已经超过框框就忽略它
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT)
      return undefined;

    //如果有炸弹 就在它的这个属性上加1
    // if (state[y2][x2].mine)
    //   block.adjacentMines += 1;

    return state.value[y2][x2] //返回位置周围的<x,y>编号
  })
    .filter(Boolean) as BlockState[]
}




//定义每个数值的颜色！
const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-teal-500',
]
// 颜色
function getBlockClass(block: BlockState) {
  if (block.flagged) {
    return 'bg-gray-500/10'
  }
  //还没翻开的时候就不给颜色
  if (!block.revealed) {
    return 'bg-gray-500/10 hover:bg-gray-500/20'
  }
  return block.mine ? 'bg-red-500/10' : numberColors[block.adjacentMines]
}

//先不生成，等第一下点击以后再生成代码
let mineGenerated = false
//点击以后的效果
function onClick(e: MouseEvent, block: BlockState) {
  if (!mineGenerated) {  //第一次点击以后再生成炸弹💣
    generateMines(block) //传点击的坐标过去！
    mineGenerated = true
  }
  expendZero(block)
  block.revealed = true //点击以后就是翻开
  if (block.mine) {
    alert('BOOOOM!')
  }
}
//右键
function onRightClick(block: BlockState) {
  if (block.revealed)
    return
  block.flagged = !block.flagged
}

//点击以后展开周围的0
function expendZero(block: BlockState) {
  if (block.adjacentMines || block.revealed) { //如果炸弹数值不为0 就直接返回
    return
  }
  //以下代码是处理 数值为0的👇 对这个点的方向进行一个循环遍历！
  getSiblings(block).forEach(s => {
    s.revealed = true
    expendZero(s)
  })
}

watchEffect(checkGameState)

//检查是否胜利
function checkGameState() {
  //如果没有还没有生成炸弹就先不要去判断
  if (!mineGenerated)
    return

  const blocks = state.value.flat()

  //  所有坐标被翻开了或者标记上🚩了才返回【true】
  if (blocks.every(block => block.revealed || block.flagged))
    //检查任何一个坐标【被标记】了并且【不是炸弹】的时候就返回 You cheat
    if (blocks.some(block => block.flagged && !block.mine))
      alert('You cheat!')
    else
      alert('You win!')
}


//在开发环境下
// let dev = true
let dev = false
</script>

<template>
  <div>
    XGGame - Minesweeper
    <div p5>
      <div v-for="row, y in state" :key="y" flex="~" items-center justify-center>
        <button v-for="block, x in row" :key="x" w-10 h-10 border="1 gray-400/10" flex="~" items-center justify-center
          m="0.5" @click="onClick($event, block)" @contextmenu.prevent="onRightClick(block)"
          :class="getBlockClass(block)">
          <template v-if="block.flagged">
            🚩
          </template>
          <template v-if="block.revealed || dev">
            <!-- 可以安装icon库【npm i -D @iconify-json/mdi】 -->
            <div v-if="block.mine"> 💣 </div>
            <div v-else> {{ block.adjacentMines }} </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>

<route lang="yaml">

</route>
