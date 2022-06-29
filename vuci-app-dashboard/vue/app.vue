<template>
<div class="devider">
  <draggable
    class="container"
    :key="refreshModal"
    v-model="allCards"
    @change="updateInfo">
       <custom-card
       class="list-item"
       v-for="item in allCards"
       :key="item.order"
       :item="item"
       />
   </draggable>
<div class="right">
      <a-button @click="setModalVisible(true)" type="primary">Filter cards</a-button>
    <filter-modal
      :isModalVisible="isModalVisible"
      :allCards="allCards"
      v-on:updateInfo="updateInfo"
      v-on:reset="reset"
      />
  </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import FilterModal from './components/FilterModal.vue'
import CustomCard from './components/CustomCard.vue'
export default {
  components: {
    draggable,
    FilterModal,
    CustomCard
  },
  data () {
    return {
      allCards: [],
      isModalVisible: false,
      lastCPUTime: null,
      cpuPercentage: 100,
      memPercentage: 100,
      sortInfoBeforeDestroy: [],
      sortInfoAfterCreated: [],
      order: [],
      sid: '',
      refreshWatch: 0,
      refreshModal: 0
    }
  },
  timers: {
    updateCpuUsage: { time: 1000, autostart: true, immediate: true, repeat: true }
  },
  methods: {
    async updateInfo () {
      await this.delSection()
      await this.saveData()
      this.setModalVisible(false)
    },
    async reset () {
      await this.delSection()
      const reset = [1, 2, 3, 4, 5]
      this.allCards.sort((a, b) => {
        return (
          reset.indexOf(a.order) - reset.indexOf(b.order)
        )
      })
      this.allCards.map(card => { card.visible = true })
      this.sortInfoAfterCreated = []
    },
    delSection () {
      if (this.sortInfoAfterCreated.length) {
        return new Promise((resolve, reject) => {
          this.sortInfoAfterCreated.map(s => {
            this.$uci.del('cardSort', s['.name'])
            resolve('promisse was resolved')
          })
        })
          .then(() => this.$uci.save())
          .then(() => this.$uci.apply())
      }
    },
    setModalVisible (boolean) {
      this.isModalVisible = boolean
    },
    async saveData () {
      await this.$uci.load('cardSort')
      this.sortInfoBeforeDestroy = this.allCards.map(card => {
        return {
          visible: card.visible ? 1 : 0, name: card.name
        }
      })
      return new Promise((resolve) => {
        this.sortInfoBeforeDestroy.map(item => {
          this.sid = this.$uci.add('cardSort', 'sort')
          this.$uci.set('cardSort', this.sid, 'visible', item.visible)
          this.$uci.set('cardSort', this.sid, 'name', item.name)
          resolve()
        })
      })
        .then(() => this.$uci.save())
        .then(() => this.$uci.apply())
    },
    updateCpuUsage () {
      this.$rpc.call('system', 'cpu_time').then(times => {
        if (!this.lastCPUTime) {
          this.cpuPercentage = 0
          this.lastCPUTime = times
          return
        }

        const idle1 = this.lastCPUTime[3]
        const idle2 = times[3]

        let total1 = 0
        let total2 = 0

        this.lastCPUTime.forEach(t => {
          total1 += t
        })

        times.forEach(t => {
          total2 += t
        })

        this.cpuPercentage = Math.floor(((total2 - total1 - (idle2 - idle1)) / (total2 - total1)) * 100)
        this.lastCPUTime = times
      })
    },
    getSystemInfo () {
      this.$system.getInfo().then(({ release, localtime, uptime, memory }) => {
        this.memPercentage = Math.floor(((memory.total - memory.free) / memory.total) * 100)
        this.allCards.push({
          name: 'System Info',
          description: [
            { title: 'Uptime', value: '%t'.format(uptime) },
            { title: 'Local time', value: new Date(localtime * 1000).toString() },
            { title: 'Firmware Version', value: release.revision }
          ],
          visible: false
        }
        )
      })
    },
    getInterfaces () {
      this.$network.load().then(() => this.$network.getInterfaces()).then((res) => {
        res.map(intrfc => {
          this.allCards.unshift({
            name: intrfc.name,
            description: [
              {
                title: 'TYPE',
                value: intrfc.status.device ? intrfc.status.device : '---'
              },
              {
                title: 'IP ADDRESS',
                value: Object.prototype.hasOwnProperty.call(intrfc.status, ['ipv4-address'][0]) ? intrfc.status['ipv4-address'][0].address : '---'
              }],
            visible: false
          })
        })
      })
    },
    getSysLogAndSort () {
      this.$rpc.call('system', 'syslog').then(({ log }) => {
        const logs = log.slice(0, 5).map((v) => {
          return { title: v.datetime, value: v.msg.slice(23) }
        })
        this.allCards.push(
          {
            name: 'system log',
            description: logs,
            visible: false
          }
        )
        this.sort()
      })
    },
    async loadConfigs () {
      this.$spin()
      await this.$uci.load('cardSort')
      this.sortInfoAfterCreated = await this.$uci.sections('cardSort', 'sort')
      this.getSystemInfo()
      this.getInterfaces()
      this.getSysLogAndSort()
    },
    async sortingOrder () {
      await this.$uci.load('cardSort')
      let res
      try {
        res = await this.$uci.sections('cardSort', 'sort').map(o => {
          return { order: o.name, visible: parseInt(o.visible) }
        })
      } catch {
        alert('sorting order failed on request')
      }
      this.order = res
      this.refreshModal++
    },
    sort () {
      const order = this.order.map(o => o.order)
      this.allCards.sort((a, b) => {
        return (
          order.indexOf(a.name) - order.indexOf(b.name)
        )
      })
      this.allCards.map((card) => {
        this.order.map(r => {
          if (r.order === card.name) {
            if (r.visible) {
              card.visible = true
              return
            }
            card.visible = false
          }
        })
      })
      this.$spin(false)
    }
  },
  async created () {
    await this.loadConfigs()
    await this.sortingOrder()
    await this.sort()
  }
}
</script>

<style scoped>
.devider {
  display: grid;
  grid-template-columns: 9fr 1fr;
}
.container {
  display: flex;
  flex-wrap: wrap;
}
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  justify-content: space-between;
}
.flex{
  display: flex;
  flex-direction: column;
}
.ant-card {
  width: 325px;
  height: 390px;
  margin-bottom: 16px;
  margin-right: 8px;
  overflow: hidden;
  text-overflow: ellipsis;
  word-break: break-all;
  transition: height 1s;
}

.ant-card:hover {
  height: auto;
  z-index: 20;
}
.ant-card h2 {
  font-size: 20px;
}

.right {
  justify-self: end;
}

.ant-modal{
  margin-right: 16px;
  width: 200px !important;
}

.ant-modal div{
  width: auto;
}

/*  */

.flip-transition-move {
  transition: all 0.7s;
}
.list-item{
  padding: 10px;
  border: 1px solid #ccc;
}
</style>
