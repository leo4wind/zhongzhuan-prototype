<script setup>
import { computed, ref } from 'vue'
import {
  AlertTriangle,
  ArrowRightLeft,
  BarChart3,
  CheckCircle2,
  ChevronsUpDown,
  Clock3,
  Filter,
  Gauge,
  MapPinned,
  PackageCheck,
  RefreshCw,
  Route,
  Search,
  Settings2,
  ShieldCheck,
  Truck,
  Warehouse
} from '@lucide/vue'
import stationFloor from './assets/station-floor.png'

const stations = [
  { id: 'HZ-N', name: '杭州北中转站', city: '杭州', load: 82, risk: '高峰', eta: '18:40' },
  { id: 'NB-E', name: '宁波东集散点', city: '宁波', load: 61, risk: '平稳', eta: '19:10' },
  { id: 'SX-C', name: '绍兴城配仓', city: '绍兴', load: 74, risk: '关注', eta: '18:55' }
]

const waveViews = {
  全链路: {
    summary: '入港、分拣、出港整体波次',
    waves: [
      { id: 'W-1800', name: '晚高峰干线波次', progress: 76, status: '执行中', volume: '12,480 件' },
      { id: 'W-1930', name: '省内城配波次', progress: 44, status: '预配载', volume: '7,260 件' },
      { id: 'W-2130', name: '跨区航空截单', progress: 18, status: '待到港', volume: '3,900 件' }
    ]
  },
  入港: {
    summary: '卸车、称重、入库缓冲波次',
    waves: [
      { id: 'IN-1745', name: '华东入港卸车', progress: 91, status: '清尾中', volume: '8,240 件' },
      { id: 'IN-1830', name: '省内到港扫描', progress: 63, status: '执行中', volume: '5,780 件' },
      { id: 'IN-1915', name: '冷链优先入库', progress: 37, status: '排队中', volume: '1,460 件' }
    ]
  },
  出港: {
    summary: '集包、装车、发车截单波次',
    waves: [
      { id: 'OUT-1845', name: '上海青浦出港', progress: 88, status: '可发车', volume: '4,920 件' },
      { id: 'OUT-1905', name: '苏州吴江改派', progress: 56, status: '待换车', volume: '3,180 件' },
      { id: 'OUT-1940', name: '南京江宁预配', progress: 29, status: '补货中', volume: '2,640 件' }
    ]
  }
}

const lanes = [
  { name: 'A01 入港卸车', rate: 92, queue: 6, tone: 'green' },
  { name: 'B08 自动分拣', rate: 86, queue: 14, tone: 'blue' },
  { name: 'C12 暂存缓冲', rate: 68, queue: 31, tone: 'amber' },
  { name: 'D05 出港装车', rate: 79, queue: 9, tone: 'teal' }
]

const exceptions = [
  { level: '紧急', title: '苏州线车辆晚到 22 分钟', desc: '建议切换 D05 月台并释放 2 名装卸员', tag: '运力' },
  { level: '关注', title: '冷链暂存区接近阈值', desc: '剩余容量 14%，预计 35 分钟后满仓', tag: '库容' },
  { level: '关注', title: '面单异常包裹积压', desc: '待复核 128 件，高于常态 31%', tag: '质检' }
]

const routes = [
  { line: '杭州 -> 上海青浦', depart: '18:45', fill: 96, dock: 'D05', state: '可发车' },
  { line: '杭州 -> 苏州吴江', depart: '19:05', fill: 83, dock: 'D02', state: '待换车' },
  { line: '杭州 -> 金华义乌', depart: '19:20', fill: 72, dock: 'D08', state: '补货中' },
  { line: '杭州 -> 南京江宁', depart: '19:40', fill: 58, dock: 'D11', state: '预配载' }
]

const selectedStationId = ref(stations[0].id)
const activeView = ref('全链路')
const query = ref('')

const selectedStation = computed(() => stations.find((station) => station.id === selectedStationId.value))
const currentWaveView = computed(() => waveViews[activeView.value])
const currentWaves = computed(() => currentWaveView.value.waves)
const filteredRoutes = computed(() => {
  const keyword = query.value.trim()
  if (!keyword) return routes
  return routes.filter((route) => `${route.line}${route.dock}${route.state}`.includes(keyword))
})
</script>

<template>
  <main class="app-shell">
    <header class="topbar">
      <div class="brand">
        <div class="brand-mark"><Warehouse :size="22" /></div>
        <div>
          <p class="eyebrow">Transfer Hub Console</p>
          <h1>中转站实时调度台</h1>
        </div>
      </div>

      <div class="station-switcher">
        <MapPinned :size="18" />
        <select v-model="selectedStationId" aria-label="选择中转站">
          <option v-for="station in stations" :key="station.id" :value="station.id">
            {{ station.name }}
          </option>
        </select>
        <ChevronsUpDown :size="16" />
      </div>

      <div class="toolbar">
        <button type="button" title="筛选"><Filter :size="18" /></button>
        <button type="button" title="刷新"><RefreshCw :size="18" /></button>
        <button type="button" title="设置"><Settings2 :size="18" /></button>
      </div>
    </header>

    <section class="hero-grid">
      <article class="metric primary">
        <span><Gauge :size="18" /> 当前负载</span>
        <strong>{{ selectedStation.load }}%</strong>
        <p>{{ selectedStation.city }}站 · {{ selectedStation.risk }} · 下一截单 {{ selectedStation.eta }}</p>
      </article>
      <article class="metric">
        <span><PackageCheck :size="18" /> 今日吞吐</span>
        <strong>46,820</strong>
        <p>较昨日 +8.4%</p>
      </article>
      <article class="metric">
        <span><Clock3 :size="18" /> 平均停留</span>
        <strong>41 分钟</strong>
        <p>目标 45 分钟内</p>
      </article>
      <article class="metric">
        <span><ShieldCheck :size="18" /> 准点率</span>
        <strong>97.2%</strong>
        <p>4 条线路需关注</p>
      </article>
    </section>

    <section class="workspace">
      <aside class="panel waves">
        <div class="panel-title">
          <div>
            <h2>波次进度</h2>
            <p>{{ currentWaveView.summary }}</p>
          </div>
          <button type="button" title="波次分析"><BarChart3 :size="17" /></button>
        </div>
        <div class="tabs" role="tablist" aria-label="视图模式">
          <button
            v-for="view in ['全链路', '入港', '出港']"
            :key="view"
            type="button"
            :class="{ active: activeView === view }"
            @click="activeView = view"
          >
            {{ view }}
          </button>
        </div>
        <div class="wave-list">
          <article v-for="wave in currentWaves" :key="wave.id" class="wave-item">
            <div>
              <strong>{{ wave.name }}</strong>
              <span>{{ wave.id }} · {{ wave.volume }}</span>
            </div>
            <b>{{ wave.status }}</b>
            <div class="progress"><i :style="{ width: `${wave.progress}%` }" /></div>
          </article>
        </div>
      </aside>

      <section class="floor-panel">
        <div class="panel-title">
          <div>
            <h2>站内流向监控</h2>
            <p>入港、分拣、暂存、出港的实时瓶颈判断</p>
          </div>
          <button class="text-action" type="button"><Route :size="17" /> 生成调拨建议</button>
        </div>
        <div class="floor-map">
          <img :src="stationFloor" alt="中转站平面流向示意图" />
          <div class="hotspot inbound">
            <Truck :size="18" />
            <span>入港 94%</span>
          </div>
          <div class="hotspot sort">
            <ArrowRightLeft :size="18" />
            <span>分拣 86%</span>
          </div>
          <div class="hotspot outbound">
            <CheckCircle2 :size="18" />
            <span>出港 79%</span>
          </div>
        </div>
        <div class="lane-grid">
          <article v-for="lane in lanes" :key="lane.name" :class="['lane-card', lane.tone]">
            <span>{{ lane.name }}</span>
            <strong>{{ lane.rate }}%</strong>
            <p>队列 {{ lane.queue }} 托</p>
          </article>
        </div>
      </section>

      <aside class="panel decisions">
        <div class="panel-title">
          <h2>异常与决策</h2>
          <AlertTriangle :size="18" />
        </div>
        <article v-for="item in exceptions" :key="item.title" class="exception-item">
          <div>
            <span :class="['level', item.level === '紧急' ? 'urgent' : 'watch']">{{ item.level }}</span>
            <b>{{ item.tag }}</b>
          </div>
          <strong>{{ item.title }}</strong>
          <p>{{ item.desc }}</p>
        </article>
      </aside>
    </section>

    <section class="route-board">
      <div class="board-header">
        <div>
          <h2>发车线路</h2>
          <p>按装载率、月台、发车状态追踪可执行线路</p>
        </div>
        <label class="search-box">
          <Search :size="17" />
          <input v-model="query" type="search" placeholder="搜索线路、月台或状态" />
        </label>
      </div>
      <div class="route-table">
        <div class="route-row head">
          <span>线路</span>
          <span>发车</span>
          <span>装载</span>
          <span>月台</span>
          <span>状态</span>
        </div>
        <div v-for="route in filteredRoutes" :key="route.line" class="route-row">
          <strong>{{ route.line }}</strong>
          <span>{{ route.depart }}</span>
          <span><i class="fill" :style="{ width: `${route.fill}%` }" />{{ route.fill }}%</span>
          <span>{{ route.dock }}</span>
          <b>{{ route.state }}</b>
        </div>
      </div>
    </section>
  </main>
</template>
