<template>
  <div class="data-table">
    <button @click.prevent="onClick" class="data-table__action">Get Data</button>
    <div v-if="!data" class="data-table__error">No Data</div>
    <div v-else class="data-table__report">
      {{ data }}
    </div>
  </div>
</template>

<script>
import { payload } from '@/mocData';
import simulateAsyncReq from '@/plugins/getDataFunc';

export default {
  data() {
    return {
      data: null,
    }
  },
  methods: {
    onClick() {
      simulateAsyncReq(payload).then((response) => {
        this.data = response;
      })
      .catch(() => {
        this.data = false;
      });
    },
  },
}
</script>

<style scoped lang="scss">
@import "DataTable.scss";
</style>
