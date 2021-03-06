<template>
  <div>
    <single-table v-if='!isMultHeaderRows'
                  :data='filterResult'
                  :columns-config='cloneColumnsConfig'
                  :record-key='recordKey'
                  :header-height='headerHeight'
                  :body-height='bodyHeight'
                  :record-height='recordHeight'
                  :render-type='renderType'
                  :header-class='headerClass'
    ></single-table>
  </div>
</template>

<script>
  import './tableHelper/requestAnimationFrameUtil';
  import SingleTable from './SingleTable';
  import _ from 'lodash';
  import {
    ID_NAME,
    SCROLL_WIDTH,
    DEFAULT_TABLE_HEIGHT,
    DEFAULT_TABLE_HEADER_HEIGHT,
    DEFAULT_TABLE_RECORD_HEIGHT,
    TABLE_TYPE_COMMON,
  } from './tableHelper/constant';
  import {getRandomStr} from './tableHelper/tableUtil';

  export default {
    components: {SingleTable},
    name: 'VueTableOptimization',
    props: {
      renderType: {
        type: String,
        default () {
          return TABLE_TYPE_COMMON;
        },
      },
      columnsConfig: {
        type: Array,
        default () {
          return [];
        },
      },
      data: {
        type: Array,
        default () {
          return [];
        },
      },
      filters: {
        type: Array,
        default () {
          return [];
        },
      },
      recordKey: {
        type: String,
        required: true,
      },
      height: {
        type: Number,
        default () {
          return DEFAULT_TABLE_HEIGHT;
        },
      },
      headerHeight: {
        type: Number,
        default () {
          return DEFAULT_TABLE_HEADER_HEIGHT;
        },
      },
      recordHeight: {
        type: Number,
        default () {
          return DEFAULT_TABLE_RECORD_HEIGHT;
        },
      },
      headerClass: {
        type: String,
        default: function () {
          return 'c-table-header__default';
        },
      },
    },
    mounted () {
      this.handleResize();
    },
    watch: {
      height: {
        handler: function (val) {
          this.bodyHeight = val - this.headerHeight;
        },
        immediate: true,
      },
      columnsConfig: {
        handler: function (val) {
          const cloneColumnsConfig = _.cloneDeep(val);
          this.buildColumnUUID(cloneColumnsConfig);
          this.cloneColumnsConfig = cloneColumnsConfig;
        },
        immediate: true,
        deep: true,
      },
      data: {
        handler: function (val) {
          this.filterResult = _.cloneDeep(val);
        },
        immediate: true,
        deep: true,
      },
    },
    data () {
      return {
        filterResult: [],
        cloneColumnsConfig: [],
        cloneColumnsRow: [],
        bodyHeight: DEFAULT_TABLE_HEIGHT - DEFAULT_TABLE_HEADER_HEIGHT,
      };
    },
    computed: {
      isMultHeaderRows: function () {
        return _.find(this.columnsConfig, function (column) {
          return column.children;
        });
      },
    },
    methods: {
      buildColumnUUID: function (columnsConfig) {
        return columnsConfig.map(item => {
          if ('children' in item) this.buildColumnUUID(item.children);
          if (!item[ID_NAME]) {
            item[ID_NAME] = getRandomStr(6);
          }
          return item;
        });
      },
      handleResize: function () {
        const tableWidth = this.$el.offsetWidth - SCROLL_WIDTH;
        let width = 0;
        let widthCount = 0;
        let columnsConfig = _.cloneDeep(this.cloneColumnsConfig);
        for (const column of columnsConfig) {
          if (column.width) {
            width += column.width;
            widthCount++;
          }
        }
        const autoWidth = (tableWidth - width) / (columnsConfig.length - widthCount);
        for (const column of columnsConfig) {
          column.cWidth = column.width ? `${column.width}px` : `${autoWidth}px`;
        }
        this.cloneColumnsConfig = columnsConfig;
      },
    },
  };
</script>

<style>

</style>
