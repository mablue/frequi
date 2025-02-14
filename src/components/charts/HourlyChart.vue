<template>
  <v-chart
    v-if="trades.length > 0"
    :options="hourlyChartOptions"
    autoresize
    :theme="getChartTheme"
  />
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator';
import { Getter } from 'vuex-class';

import ECharts from 'vue-echarts';

import 'echarts/lib/chart/bar';
import 'echarts/lib/chart/line';
import 'echarts/lib/component/title';
import 'echarts/lib/component/tooltip';
import 'echarts/lib/component/legend';
import 'echarts/lib/component/visualMap';
import 'echarts/lib/component/visualMapPiecewise';

import { Trade } from '@/types';
import { timestampHour } from '@/shared/formatters';
import { EChartOption } from 'echarts';

// Define Column labels here to avoid typos
const CHART_PROFIT = 'Profit %';
const CHART_TRADE_COUNT = 'Trade Count';

@Component({
  components: {
    'v-chart': ECharts,
  },
})
export default class HourlyChart extends Vue {
  @Prop({ required: true }) trades!: Trade[];

  @Prop({ default: true, type: Boolean }) showTitle!: boolean;

  @Getter getChartTheme!: string;

  get hourlyData() {
    const res = new Array(24);
    for (let i = 0; i < 24; i += 1) {
      res[i] = { hour: i, hourDesc: `${i}h`, profit: 0.0, count: 0.0 };
    }

    for (let i = 0, len = this.trades.length; i < len; i += 1) {
      const trade = this.trades[i];
      if (trade.close_timestamp) {
        const hour = timestampHour(trade.close_timestamp);

        res[hour].profit += trade.profit_ratio;
        res[hour].count += 1;
      }
    }
    return res;
  }

  get hourlyChartOptions(): EChartOption {
    return {
      title: {
        text: 'Hourly Profit',
        show: this.showTitle,
      },
      dataset: {
        dimensions: ['hourDesc', 'profit', 'count'],
        source: this.hourlyData,
      },
      tooltip: {
        trigger: 'axis',
        axisPointer: {
          type: 'line',
          label: {
            backgroundColor: '#6a7985',
          },
        },
      },
      legend: {
        data: [CHART_PROFIT, CHART_TRADE_COUNT],
        right: '5%',
      },
      xAxis: {
        type: 'category',
      },
      yAxis: [
        {
          type: 'value',
          name: CHART_PROFIT,
          splitLine: {
            show: false,
          },
          nameRotate: 90,
          nameLocation: 'middle',
          nameGap: 30,
        },
        {
          type: 'value',
          name: CHART_TRADE_COUNT,
          nameRotate: 90,
          nameLocation: 'middle',
          nameGap: 30,
        },
      ],
      visualMap: [
        {
          dimension: 1,
          seriesIndex: 0,
          show: false,
          pieces: [
            {
              max: 0.0,
              min: -2,
              color: 'red',
            },
            {
              min: 0.0,
              max: 2,
              color: 'green',
            },
          ],
        },
      ],
      series: [
        {
          type: 'line',
          name: CHART_PROFIT,
          animation: false,
          // symbol: 'none',
        },
        {
          type: 'bar',
          name: CHART_TRADE_COUNT,
          animation: false,
          itemStyle: {
            color: 'rgba(150,150,150,0.3)',
          },
          yAxisIndex: 1,
        },
      ],
    };
  }
}
</script>

<style scoped>
.echarts {
  width: 100%;
  height: 100%;
}
</style>
