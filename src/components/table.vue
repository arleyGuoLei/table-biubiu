<template>
  <div style="height: 100vh;" @mousedown="handlerClose">
    <table id="table" @mousedown.left="downAction" @mouseup.left="upAction" @contextmenu="menuShow">
      <tr v-for="(r,index) in tableData" :key="index">
        <td
          v-for="(item,tdIndex) in r"
          :key="tdIndex"
          contenteditable
          :data-x="item.x"
          :data-y="item.y"
          :class="[item.select ? 'select' : '']"
          @mousemove="moveAction"
          @input="inputAction"
        />
      </tr>
    </table>
    <Menu
      id="menu"
      :top="menuTop"
      :left="menuLeft"
      @deleteRow="deleteRow"
      @InsertColumn="InsertColumn"
      @deleteColumn="deleteColumn"
      @InsertRow="InsertRow"
      @mergeTd="mergeTd"
    />
  </div>
</template>

<script>
import throttle from 'lodash.throttle'
import Menu from './Menu'

export default {
  name: 'Table',
  components: {
    Menu
  },
  data() {
    return {
      tableData: [], // colspan rowspan value x y // onmousedown onmousemove onmouseup
      row: '10',
      column: '10',
      record: false,
      /**
       * 选中的坐标
       */
      position: {
        x: {
          from: -1,
          to: -1
        },
        y: {
          from: -1,
          to: -1
        }
      },
      menuTop: -800,
      menuLeft: -800
    }
  },
  watch: {
    position: {
      handler(newName, oldName) {
        this.tableData = this.makeTableData()
      },
      immediate: true,
      deep: true
    }
  },
  created() {
    this.tableData = this.makeTableData()
  },
  mounted() {
  },
  destroyed() {
  },
  methods: {
    deleteRow() {
      const { from, to } = this.position.x
      this.tableData.splice(Math.min(from, to), (Math.abs(to - from) + 1))
      this.row = this.row - (Math.abs(to - from) + 1)
    },
    deleteColumn() {

    },
    InsertRow() {

    },
    InsertColumn() {

    },
    mergeTd() {

    },
    menuShow(event) {
      this.menuTop = event.clientY
      this.menuLeft = event.clientX
      event.preventDefault()
      return false
    },
    /**
     * 取消选中区域的选中
     */
    handlerClose(event) {
      if (document.getElementById('table').contains(event.target) || document.getElementById('menu').contains(event.target)) {
        return ''
      }
      this.position.x.from = -1
      this.position.y.from = -1
      this.position.x.to = -1
      this.position.y.to = -1
    },
    moveAction: throttle(function(event) {
      if (this.record) {
        const { x, y } = event.target.dataset
        this.position.x.to = parseInt(x)
        this.position.y.to = parseInt(y)
      }
    }, 100),
    /**
     * 初始化选中数据
     */
    downAction(event) {
      this.record = true
      this.position.x.from = parseInt(event.target.dataset.x)
      this.position.y.from = parseInt(event.target.dataset.y)
      this.position.x.to = parseInt(event.target.dataset.x)
      this.position.y.to = parseInt(event.target.dataset.y)
    },
    upAction(event) {
      this.record = false
    },
    /**
     * 输入完成
     */
    inputAction(event) {
      const { dataset: { x, y }, innerText } = event.target
      this.tableData[x][y].value = innerText
    },
    makeTableData() {
      const { row, column } = this
      const tableData = []
      for (let r = 0; r < parseInt(row); r++) {
        const arr = []
        for (let col = 0; col < parseInt(column); col++) {
          const select = this.inRange(r, col)
          const obj = {
            value: '',
            x: r,
            y: col,
            select
          }
          arr.push(obj)
        }
        tableData.push(arr)
      }
      return tableData
    },
    inRange(x, y) {
      const { x: { from: xFrom, to: xTo }, y: { from: yFrom, to: yTo }} = this.position
      /**
       * 左上到右下
       */
      if ((x >= xFrom && x <= xTo) && (y >= yFrom && y <= yTo)) {
        return true
      }
      /**
       * 右下到左上
       */
      if ((x <= xFrom && x >= xTo) && (y <= yFrom && y >= yTo)) {
        return true
      }
      /**
       * 左下到右上
       */
      if ((x <= xFrom && x >= xTo) && (y >= yFrom && y <= yTo)) {
        return true
      }
      /**
       * 右上 到 左下
       */
      if ((x >= xFrom && x <= xTo) && (y <= yFrom && y >= yTo)) {
        return true
      }
      return false
    }
  }

}
</script>

<style scoped>
table {
  border-collapse: collapse;
  margin: 0 auto;
  text-align: center;
}
table:active,table:hover{
  border: 1px solid #cad9ea;
}
table td,
table th {
  border: 1px solid #cad9ea;
  color: #666;
  height: 30px;
  user-select: none;
}
table td {
  padding: 0 30px;
}
table tr:nth-child(odd) {
  background: #fff;
}
table tr:nth-child(even) {
  background: #f5fafa;
}
.select{
  background:#FFFFCC;
}
</style>
