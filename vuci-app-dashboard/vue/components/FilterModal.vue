<template>
  <div>
    <a-modal :visible="isModalVisible" :closable="false" title="Filter cards">
    <a-row v-for="check in allCards" :key="check.name">
      <a-checkbox v-model="check.visible"><b>{{check.name}}</b></a-checkbox>
    </a-row>
      <template #footer>
        <a-button type="primary" key="back" @click="updateInfo">OK</a-button>
        <a-button type="secondary" @click="reset">Reset</a-button>
      </template>
    </a-modal>
  </div>
</template>

<script>
export default {
  name: 'FilterModal',
  props: ['isModalVisible', 'allCards'],
  data () {
    return {
    }
  },
  methods: {
    updateInfo () {
      this.$emit('updateInfo')
    },
    reset () {
      this.$emit('reset')
    }
  }
}
</script>

<style scoped>
.flex {
  display: flex;
  width: 30%;
  margin: 0 auto;
}

button {
  margin-left: 8px;
}

.mainForm .ant-form-item-children div {
  display: none;
}

.ant-modal-footer {
  display: none;
}
</style>
