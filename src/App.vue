<script setup>
import { computed, ref } from 'vue'
import { dashboard, employees, moduleDefinitions, modulePages, secondaryViews } from './mockData'

const primaryViews = [
  { id: 'dashboard', label: 'HR 儀表板', caption: '數據總覽' },
  { id: 'profile', label: '員工基本資料', caption: '主檔管理' },
  ...secondaryViews,
]

const activeView = ref('dashboard')
const selectedEmployeeId = ref(employees[0].id)

const selectedEmployee = computed(
  () => employees.find((employee) => employee.id === selectedEmployeeId.value) || employees[0],
)

const activeModules = computed(() =>
  moduleDefinitions.map((module) => ({
    ...module,
    ...selectedEmployee.value.modules[module.id],
  })),
)

const profileStats = computed(() => [
  { label: '資料完整率', value: `${selectedEmployee.value.profileCompleteness}%`, tone: 'good' },
  { label: '待辦事項', value: selectedEmployee.value.pendingTasks, tone: 'warn' },
  { label: '最近異動', value: selectedEmployee.value.lastChangeShort, tone: 'cool' },
])

const dashboardKpis = computed(() => [
  { label: '員工總數', value: dashboard.totalEmployees, caption: '跨部門統一主檔' },
  { label: '本月異動', value: dashboard.monthlyChanges, caption: '升遷、調任、薪資調整' },
  { label: '即時到班率', value: `${dashboard.attendanceRate}%`, caption: '已排除請假與補登' },
  { label: '薪資檢核', value: `${dashboard.payrollCheck}%`, caption: '月結前檢查進度' },
])

const placeholderView = computed(() =>
  secondaryViews.find((view) => view.id === activeView.value) || secondaryViews[0],
)

const activeModulePage = computed(() => modulePages[activeView.value] || modulePages.contract)

function isNavActive(viewId) {
  if (viewId === 'profile' && activeView.value === 'detail') return true
  return activeView.value === viewId
}

function selectEmployee(id) {
  selectedEmployeeId.value = id
  activeView.value = 'profile'
}

function openProfileDetail() {
  activeView.value = 'detail'
}

function activateView(viewId) {
  activeView.value = viewId
}
</script>

<template>
  <div class="aurora"></div>
  <div class="app-shell">
    <aside class="sidebar">
      <div class="brand">
        <span class="brand-mark">HR</span>
        <div>
          <strong>PTS HRMS</strong>
          <p>Nordic Control</p>
        </div>
      </div>

      <nav class="nav-list" aria-label="Demo views">
        <button
          v-for="view in primaryViews"
          :key="view.id"
          :class="['nav-item', { active: isNavActive(view.id) }]"
          type="button"
          @click="activateView(view.id)"
        >
          <span>{{ view.label }}</span>
          <small>{{ view.caption }}</small>
        </button>
      </nav>

      <div class="sync-card">
        <span class="live-dot"></span>
        <div>
          <strong>AD / ERP / 考勤門禁</strong>
          <p>Mock Online</p>
        </div>
      </div>
    </aside>

    <main class="workspace">
      <header class="topbar">
        <div>
          <p class="eyebrow">財團法人公共電視文化事業基金會</p>
          <h1>人員管理</h1>
        </div>
        <div class="topbar-meta">
          <span>人資專員</span>
          <strong>{{ selectedEmployee.name }}</strong>
        </div>
      </header>

      <section v-if="activeView === 'dashboard'" class="dashboard-view">
        <div class="dashboard-head">
          <div class="section-heading">
            <p>HR Dashboard</p>
            <h2>數據分析與報表</h2>
          </div>
          <div class="dashboard-chip-group">
            <span class="dashboard-chip">人力成本</span>
            <span class="dashboard-chip">流動率</span>
            <span class="dashboard-chip">到班狀態</span>
          </div>
        </div>

        <div class="kpi-grid">
          <article v-for="kpi in dashboardKpis" :key="kpi.label" class="kpi-card">
            <span>{{ kpi.label }}</span>
            <strong>{{ kpi.value }}</strong>
            <p>{{ kpi.caption }}</p>
          </article>
        </div>

        <div class="chart-grid">
          <article class="chart-panel wide">
            <div class="section-heading">
              <p>Cost Trend</p>
              <h2>人力成本與薪資趨勢</h2>
            </div>
            <svg class="area-chart" viewBox="0 0 640 260" role="img" aria-label="人力成本趨勢圖">
              <defs>
                <linearGradient id="areaFill" x1="0" x2="0" y1="0" y2="1">
                  <stop offset="0%" stop-color="#6cc7c2" stop-opacity="0.66" />
                  <stop offset="100%" stop-color="#6cc7c2" stop-opacity="0.04" />
                </linearGradient>
              </defs>
              <path class="grid-line" d="M40 40H610M40 100H610M40 160H610M40 220H610" />
              <path class="area-fill" d="M40 196 L130 174 L220 150 L310 164 L400 106 L500 84 L610 56 L610 226 L40 226 Z" />
              <path class="area-line" d="M40 196 L130 174 L220 150 L310 164 L400 106 L500 84 L610 56" />
              <g class="chart-dots">
                <circle cx="40" cy="196" r="5" />
                <circle cx="130" cy="174" r="5" />
                <circle cx="220" cy="150" r="5" />
                <circle cx="310" cy="164" r="5" />
                <circle cx="400" cy="106" r="5" />
                <circle cx="500" cy="84" r="5" />
                <circle cx="610" cy="56" r="5" />
              </g>
            </svg>
          </article>

          <article class="chart-panel">
            <div class="section-heading">
              <p>Turnover</p>
              <h2>部門流動率</h2>
            </div>
            <div class="bar-chart">
              <div v-for="bar in dashboard.turnover" :key="bar.label" class="bar-row">
                <span>{{ bar.label }}</span>
                <div><b :style="{ width: `${bar.value}%` }"></b></div>
                <strong>{{ bar.value }}%</strong>
              </div>
            </div>
          </article>

          <article class="chart-panel">
            <div class="section-heading">
              <p>Attendance</p>
              <h2>即時到班狀態</h2>
            </div>
            <div class="donut-wrap">
              <svg viewBox="0 0 140 140" class="donut-chart" aria-label="即時到班率">
                <circle cx="70" cy="70" r="52" />
                <circle cx="70" cy="70" r="52" :style="{ strokeDashoffset: 327 - dashboard.attendanceRate * 3.27 }" />
              </svg>
              <div>
                <strong>{{ dashboard.attendanceRate }}%</strong>
                <span>On Site / Remote</span>
              </div>
            </div>
            <div class="attendance-list">
              <span v-for="item in dashboard.attendance" :key="item.label">
                <b :class="item.tone"></b>{{ item.label }} {{ item.value }}
              </span>
            </div>
          </article>
        </div>
      </section>

      <section v-else-if="activeView === 'profile'" class="view-grid profile-view">
        <aside class="directory">
          <div class="section-heading">
            <p>Employee Directory</p>
            <h2>員工清單</h2>
          </div>
          <button
            v-for="employee in employees"
            :key="employee.id"
            :class="['employee-row', { active: employee.id === selectedEmployee.id }]"
            type="button"
            @click="selectEmployee(employee.id)"
          >
            <span class="avatar">{{ employee.initials }}</span>
            <span>
              <strong>{{ employee.name }}</strong>
              <small>{{ employee.department }} · {{ employee.title }}</small>
            </span>
            <b>{{ employee.risk }}</b>
          </button>
        </aside>

        <section class="profile-main">
          <article class="hero-profile">
            <div class="employee-hero">
              <span class="avatar large">{{ selectedEmployee.initials }}</span>
              <div>
                <p class="eyebrow">Basic Profile</p>
                <h2>{{ selectedEmployee.name }}</h2>
                <p>{{ selectedEmployee.department }} · {{ selectedEmployee.title }}</p>
              </div>
              <div class="hero-actions">
                <span class="status-pill">{{ selectedEmployee.status }}</span>
                <button class="detail-button" type="button" @click="openProfileDetail">查看資料</button>
              </div>
            </div>

            <div class="profile-fields">
              <div>
                <span>員工編號</span>
                <strong>{{ selectedEmployee.id }}</strong>
              </div>
              <div>
                <span>主管</span>
                <strong>{{ selectedEmployee.manager }}</strong>
              </div>
              <div>
                <span>入職日期</span>
                <strong>{{ selectedEmployee.hireDate }}</strong>
              </div>
              <div>
                <span>年資</span>
                <strong>{{ selectedEmployee.tenure }}</strong>
              </div>
              <div>
                <span>契約型態</span>
                <strong>{{ selectedEmployee.contractType }}</strong>
              </div>
              <div>
                <span>排班型態</span>
                <strong>{{ selectedEmployee.shiftType }}</strong>
              </div>
            </div>
          </article>

          <div class="stat-strip">
            <article v-for="stat in profileStats" :key="stat.label" :class="['stat-card', stat.tone]">
              <span>{{ stat.label }}</span>
              <strong>{{ stat.value }}</strong>
            </article>
          </div>

          <section class="module-grid">
            <article v-for="module in activeModules" :key="module.id" class="module-card">
              <span class="module-icon">{{ module.icon }}</span>
              <div>
                <p>{{ module.label }}</p>
                <h3>{{ module.value }}</h3>
                <small>{{ module.detail }}</small>
              </div>
            </article>
          </section>
        </section>

        <aside class="timeline-panel">
          <div class="section-heading">
            <p>Recent Changes</p>
            <h2>異動歷程</h2>
          </div>
          <div class="timeline">
            <div v-for="item in selectedEmployee.timeline" :key="item.date + item.title" class="timeline-item">
              <span>{{ item.date }}</span>
              <div>
                <strong>{{ item.title }}</strong>
                <p>{{ item.detail }}</p>
              </div>
            </div>
          </div>
        </aside>
      </section>

      <section v-else-if="activeView === 'detail'" class="view-grid detail-view">
        <article class="detail-stage">
          <div class="section-heading">
            <p>Detail View</p>
            <h2>{{ selectedEmployee.name }} 詳細資料</h2>
          </div>

          <div class="detail-summary">
            <div v-for="card in selectedEmployee.detailCards" :key="card.label" class="detail-card">
              <span>{{ card.label }}</span>
              <strong>{{ card.value }}</strong>
              <p>{{ card.detail }}</p>
            </div>
          </div>

          <div class="detail-grid">
            <article v-for="module in activeModules" :key="module.id" class="pipeline-node">
              <span>{{ module.icon }}</span>
              <strong>{{ module.label }}</strong>
              <p>{{ module.detail }}</p>
            </article>
          </div>
        </article>

        <aside class="detail-panel">
          <div class="section-heading">
            <p>Related Records</p>
            <h2>相關紀錄</h2>
          </div>
          <div class="audit-list">
            <div v-for="event in selectedEmployee.records" :key="event.time + event.title" class="audit-event">
              <span>{{ event.time }}</span>
              <div>
                <strong>{{ event.title }}</strong>
                <p>{{ event.detail }}</p>
              </div>
            </div>
          </div>
        </aside>
      </section>

      <section v-else class="module-page">
        <article class="placeholder-hero">
          <div class="section-heading">
            <p>{{ activeModulePage.eyebrow }}</p>
            <h2>{{ activeModulePage.title }}</h2>
          </div>
          <p>{{ placeholderView.description }}</p>
          <div class="placeholder-tags">
            <span v-for="tag in placeholderView.tags" :key="tag">{{ tag }}</span>
          </div>
        </article>

        <div class="module-kpi-grid">
          <article v-for="item in activeModulePage.summary" :key="item.label" class="placeholder-card">
            <span>{{ item.label }}</span>
            <strong>{{ item.value }}</strong>
            <p>{{ item.caption }}</p>
          </article>
        </div>

        <div class="module-layout">
          <article class="module-table-card">
            <div class="section-heading">
              <p>Main Workspace</p>
              <h2>{{ activeModulePage.title }}總覽</h2>
            </div>
            <div class="table-wrap">
              <table class="data-table">
                <thead>
                  <tr>
                    <th v-for="column in activeModulePage.columns" :key="column">{{ column }}</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="row in activeModulePage.rows" :key="row.join('-')">
                    <td v-for="cell in row" :key="cell">{{ cell }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </article>

          <aside class="module-side-panel">
            <div class="section-heading">
              <p>Focus</p>
              <h2>{{ activeModulePage.sideTitle }}</h2>
            </div>
            <div class="module-highlight-list">
              <article
                v-for="item in activeModulePage.highlights"
                :key="item.title"
                :class="['module-highlight-card', item.tone]"
              >
                <strong>{{ item.title }}</strong>
                <p>{{ item.detail }}</p>
              </article>
            </div>
          </aside>
        </div>

        <div class="placeholder-grid">
          <article class="placeholder-card slim">
            <span>Quick Action</span>
            <strong>查詢條件</strong>
            <p>依部門、日期、狀態與負責人快速切換資料視角。</p>
          </article>
          <article class="placeholder-card slim">
            <span>Current Data</span>
            <strong>Mock Records</strong>
            <p>這一版先固定資料，讓你簡報切頁時有完整內容。</p>
          </article>
          <article class="placeholder-card slim">
            <span>Next Step</span>
            <strong>補互動</strong>
            <p>後續可再接表單編修、匯出、批次流程與權限細節。</p>
          </article>
        </div>
      </section>
    </main>
  </div>
</template>
