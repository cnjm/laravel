<template>
  <div>
    <div class="items">
      <div class="item">
        <el-input label="a" v-model="input" placeholder="请输入内容1">
          <template slot="prepend">字段名</template>
        </el-input>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        input:"",
      }
    }
  }
</script>

<style scoped>
  .title {
    color: #888;
    font-size: 18px;
    font-weight: initial;
    letter-spacing: .25px;
    margin-top: 10px;
  }

  .items { margin-top: 8px; }

  .item {
    display: flex;
    margin-bottom: 6px;
  }

  .item .name {
    color: #6a6a6a;
    margin-right: 6px;
  }

  .item .value {
    color: #35495e;
    font-weight: bold;
  }
</style>
