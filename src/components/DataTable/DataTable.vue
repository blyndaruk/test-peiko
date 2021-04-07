<template>
  <div class="data-table">
    <button @click.prevent="onClick" class="data-table__action" :class="{ 'is-loading': isLoading }">
      <VLoader v-if="isLoading" />
      <span v-else>Get Data</span>
    </button>
    <div v-if="!data" class="data-table__error">No Data</div>
    <div v-else class="data-table__report">
      <div class="report">
        <div class="report__head">
          <div class="report__th">Stock</div>
          <div class="report__th">Current</div>
          <div class="report__th">Change</div>
        </div>
        <div class="report__body">
          <div class="report__row" v-for="(stock, index) in report.stocks" :key="stock">
            <div class="report__cell" v-if="report.stocks[index]">{{ report.stocks[index] }}</div>
            <div class="report__cell" v-if="report.current[index]">{{ formatNumber(report.current[index]) }}</div>
            <div
                v-if="report.current[index]"
                :class="negative( change(report.start[index], report.current[index]) ) ? 'is-negative' : 'is-positive'"
                class="report__cell"
            >
              {{ change(report.start[index], report.current[index]) }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { payload } from '@/mocData';
import simulateAsyncReq from '@/plugins/getDataFunc';

import VLoader from '@/components/VLoader/VLoader';

export default {
  components: { VLoader },
  data() {
    return {
      data: null,
      isLoading: false,
    }
  },
  methods: {
    onClick() {
      this.isLoading = true;
      simulateAsyncReq(payload).then((response) => {
        this.data = response;
        this.isLoading = false;
      })
      .catch(() => {
        this.data = false;
        this.isLoading = false;
      });
    },
    /**
     *
     * @param arrays
     * @param comparator
     * @returns {{}|*[]}
     */
    parallelSorting(arrays, comparator = (a, b) => (a < b) ? -1 : (a > b) ? 1 : 0) {
      const arrayKeys = Object.keys(arrays);
      const sortableArray = Object.values(arrays)[0];
      const indexes = Object.keys(sortableArray);
      const sortedIndexes = indexes.sort((a, b) => comparator(sortableArray[a], sortableArray[b]));

      let sortByIndexes = (array, sortedIndexes) => sortedIndexes.map(sortedIndex => array[sortedIndex]);

      if (Array.isArray(arrays)) {
        return arrayKeys.map(arrayIndex => sortByIndexes(arrays[arrayIndex], sortedIndexes));
      } else {
        let sortedArrays = {};
        arrayKeys.forEach((arrayKey) => {
          sortedArrays[arrayKey] = sortByIndexes(arrays[arrayKey], sortedIndexes);
        });
        return sortedArrays;
      }
    },
    formatNumber(value) {
      return parseFloat(value).toFixed(2)
    },
    negative(value) {
      return value < 0;
    },
    change(start, current) {
      const diff = this.formatNumber( (parseFloat(current) - parseFloat(start)) )
      const sign = diff < 0 ? '' : '+';
      return sign + diff;
    },
  },
  computed: {
    report() {
      if (!this.data) return;
      return this.parallelSorting(this.data)
    }
  }
}
</script>

<style scoped lang="scss">
@import "DataTable.scss";
</style>
