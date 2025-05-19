<template>
  <div class="p-8 max-w-6xl mx-auto space-y-10 rounded-xl bg-gradient-to-br from-indigo-100 via-sky-50 to-teal-100 shadow-md">
    <h1 class="text-4xl font-bold text-center text-blue-700 drop-shadow-sm">E-Rostering Demand Calculator</h1>

    <!-- Template Management -->
    <div class="border border-blue-200 p-4 rounded-lg bg-white mb-6">
      <h2 class="text-xl font-semibold mb-2">Templates</h2>
      <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 items-end mb-4">
        <input
          v-model="newTemplateName"
          placeholder="Template Name"
          class="field"
          aria-label="Template Name"
        />
        <button @click="saveCurrentTemplate" class="action" aria-label="Save Template">Save Template</button>
        <select v-model="selectedTemplateIndex" class="field" aria-label="Select Template">
          <option disabled value="">Select Template</option>
          <option v-for="(t, i) in templates" :key="i" :value="i">{{ t.name }}</option>
        </select>
        <button
          @click="loadTemplate(templates[selectedTemplateIndex])"
          :disabled="selectedTemplateIndex === ''"
          class="action"
          aria-label="Load Template"
        >
          Load Template
        </button>
      </div>
      <div class="flex justify-end space-x-4">
        <button @click="exportCSV" class="action" aria-label="Export CSV">Export CSV</button>
        <button @click="exportPDF" class="action" aria-label="Export PDF">Export PDF</button>
      </div>
    </div>

    <!-- Shift Configuration -->
    <details
      id="shift-section"
      role="region"
      aria-labelledby="shift-summary"
      open
      class="bg-white border border-blue-200 p-6 rounded-lg shadow-md transition-all duration-300 hover:shadow-lg"
    >
      <summary id="shift-summary" class="mb-4 cursor-pointer select-none">
        <button
          type="button"
          class="w-full text-left text-2xl font-semibold text-indigo-600 flex items-center gap-2 focus:outline-none"
          aria-expanded="true"
          aria-controls="shift-section"
        >
          <span class="material-symbols-outlined" aria-hidden="true">schedule</span>
          <span>Configure Shifts</span>
        </button>
      </summary>
      <transition-group name="fade" tag="div">
        <div
          v-for="(shift, index) in shifts"
          :key="shift.id"
          class="grid grid-cols-1 sm:grid-cols-5 gap-4 mb-3 items-center"
        >
          <div class="flex flex-col">
            <label class="text-sm font-semibold text-gray-700 mb-1">Shift Name</label>
            <input
              v-model="shift.name"
              placeholder="e.g. Morning"
              class="field"
              aria-label="Shift Name"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-sm font-semibold text-gray-700 mb-1">Start Time</label>
            <input
              v-model="shift.start"
              type="time"
              step="60"
              class="field"
              aria-label="Start Time"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-sm font-semibold text-gray-700 mb-1">End Time</label>
            <input
              v-model="shift.end"
              type="time"
              step="60"
              class="field"
              aria-label="End Time"
            />
          </div>

          <div class="flex flex-col">
            <label class="text-sm font-semibold text-gray-700 mb-1">Break (mins)</label>
            <input
              v-model.number="shift.break"
              type="number"
              placeholder="30"
              class="field"
              aria-label="Break Length"
            />
          </div>

          <div class="flex flex-col justify-end">
            <button
              @click="removeShift(index)"
              class="action"
              aria-label="Remove Shift"
            >
              Remove
            </button>
          </div>
        </div>
      </transition-group>
      <button @click="addShift" class="action" aria-label="Add Shift">Add Shift</button>
    </details>

    <!-- Role Configuration -->
    <details
      id="role-section"
      role="region"
      aria-labelledby="role-summary"
      open
      class="bg-white border border-blue-200 p-6 rounded-lg shadow-md transition-all duration-300 hover:shadow-lg"
    >
      <summary id="role-summary" class="mb-4 cursor-pointer select-none">
        <button
          type="button"
          class="w-full text-left text-2xl font-semibold text-teal-600 flex items-center gap-2 focus:outline-none"
          aria-expanded="true"
          aria-controls="role-section"
        >
          <span class="material-symbols-outlined" aria-hidden="true">groups</span>
          <span>Configure Roles</span>
        </button>
      </summary>
      <transition-group name="fade" tag="div">
        <div
          v-for="(role, roleIndex) in roles"
          :key="role.id"
          class="border p-4 rounded-lg bg-blue-50 mb-6"
        >
          <label class="block text-sm font-semibold text-gray-700">Role Name</label>
          <input
            v-model="role.name"
            placeholder="e.g. Nurse"
            class="field"
            aria-label="Role Name"
          />
          <label class="block text-sm font-semibold text-gray-700">Whole Time Equivalent (WTE)</label>
          <input
            v-model.number="role.wte"
            type="number"
            placeholder="1.0"
            class="field mb-4"
            aria-label="WTE"
          />

          <transition-group name="fade" tag="div">
            <div
              v-for="(vt, vtIndex) in role.visitTypes"
              :key="vt.id"
              class="grid grid-cols-5 sm:grid-cols-1 gap-2 mb-3 items-center"
            >
              <label class="block text-sm font-semibold text-gray-700">Visit Type</label>
              <input
                v-model="vt.name"
                placeholder="e.g. Home Visit"
                class="field"
                aria-label="Visit Type"
              />

              <label class="block text-sm font-semibold text-gray-700">Visit Length (mins)</label>
              <input
                v-model.number="vt.length"
                type="number"
                placeholder="30"
                class="field"
                aria-label="Visit Length"
              />

              <label class="block text-sm font-semibold text-gray-700">Visits per Shift</label>
              <input
                v-model.number="vt.count"
                type="number"
                placeholder="5"
                class="field"
                aria-label="Visits per Shift"
              />

              <label class="block text-sm font-semibold text-gray-700">Assigned Shift</label>
              <select
                v-model.number="vt.shiftIndex"
                class="field"
                aria-label="Assigned Shift"
              >
                <option v-for="(s, idx) in shifts" :value="idx">{{ s.name }}</option>
              </select>

              <button
                @click="removeVisitType(role, vtIndex)"
                class="action"
                aria-label="Remove Visit Type"
              >
                Remove
              </button>
            </div>
          </transition-group>

          <div class="flex flex-col space-y-2">
            <button
              @click="addVisitType(role)"
              class="action"
              aria-label="Add Visit Type"
            >
              Add Visit Type
            </button>
            <button
              @click="removeRole(roleIndex)"
              class="action"
              aria-label="Remove Role"
            >
              Remove Role
            </button>
          </div>
        </div>
      </transition-group>
      <button @click="addRole" class="action" aria-label="Add Role">Add Role</button>
    </details>

    <!-- Travel Settings -->
    <details
      id="travel-section"
      role="region"
      aria-labelledby="travel-summary"
      open
      class="bg-white border border-blue-200 p-6 rounded-lg shadow-md transition-all duration-300 hover:shadow-lg"
    >
      <summary id="travel-summary" class="mb-4 cursor-pointer select-none">
        <button
          type="button"
          class="w-full text-left text-2xl font-semibold text-indigo-600 flex items-center gap-2 focus:outline-none"
          aria-expanded="true"
          aria-controls="travel-section"
        >
          <span class="material-symbols-outlined" aria-hidden="true">map</span>
          <span>Travel Settings</span>
        </button>
      </summary>
      <label
        class="block mb-2 text-gray-700 font-medium"
        aria-label="Average Travel Time"
      >
        Average Travel Time (mins)
      </label>
      <input
        v-model.number="averageTravelTime"
        type="number"
        placeholder="10"
        class="field"
        aria-label="Average Travel Time"
      />
    </details>

    <!-- Demand Summary -->
    <details
      id="summary-section"
      role="region"
      aria-labelledby="summary-summary"
      open
      class="bg-white border border-blue-300 p-6 rounded-lg shadow-md transition-all duration-300 hover:shadow-lg"
    >
      <summary id="summary-summary" class="mb-4 cursor-pointer select-none">
        <button
          type="button"
          class="w-full text-left text-2xl font-bold text-cyan-700 flex items-center gap-2 focus:outline-none"
          aria-expanded="true"
          aria-controls="summary-section"
        >
          <span class="material-symbols-outlined" aria-hidden="true">calculate</span>
          <span>Demand Summary</span>
        </button>
      </summary>

      <div v-if="warningMessage" class="text-red-600 font-medium mb-4" role="alert" aria-live="assertive">
        {{ warningMessage }}
      </div>
      <div v-if="demandOutput.length" class="space-y-6">
        <div
          v-for="role in demandOutput"
          :key="role.name"
          class="border p-4 rounded-lg bg-cyan-50"
        >
          <h3 class="text-xl font-semibold text-blue-800 mb-2">{{ role.name }}</h3>
          <p
            class="text-sm font-medium mb-2"
            :class="{
              'text-green-600': role.wteStatus.includes('Under'),
              'text-red-600': role.wteStatus.includes('Over'),
              'text-blue-600': role.wteStatus.includes('Matches')
            }"
          >
            WTE Required:
            <span aria-label="WTE required amount">{{ role.wteRequired }}</span>
            (<span class="italic" aria-label="WTE status">{{ role.wteStatus }}</span>)
          </p>
          <div class="grid grid-cols-2 md:grid-cols-3 gap-4 text-gray-800">
            <p>
              <strong>Total Visits:</strong>
              <span aria-label="Total visits">{{ role.totalVisits }}</span>
            </p>
            <p>
              <strong>Total Time:</strong>
              <span aria-label="Total time in hours">{{ role.totalHours }} hrs</span>
            </p>
            <p>
              <strong>Total Shifts:</strong>
              <span aria-label="Total shifts">{{ role.totalShifts }}</span>
            </p>
          </div>
          <div v-if="role.shiftCounts" class="mt-4">
            <h4 class="font-medium">Shift Breakdown:</h4>
            <ul class="list-disc list-inside ml-4">
              <li v-for="(count, shiftName) in role.shiftCounts" :key="shiftName">
                {{ shiftName }}: {{ count }} shift<span v-if="count>1">s</span>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </details>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'
import debounce from 'lodash/debounce'
import { jsPDF } from 'jspdf'

let uid = 0
const createId = () => uid++

const roles = ref([])
const shifts = ref([])
const averageTravelTime = ref(10)
const warningMessage = ref('')
const demandOutput = ref([])

// Template management
const templates = ref([])
const newTemplateName = ref('')
const selectedTemplateIndex = ref('')

const loadTemplatesFromStorage = () => {
  const stored = localStorage.getItem('templates')
  templates.value = stored ? JSON.parse(stored) : []
}
const saveTemplatesToStorage = () => {
  localStorage.setItem('templates', JSON.stringify(templates.value))
}
const saveCurrentTemplate = () => {
  if (!newTemplateName.value.trim()) return
  templates.value.push({
    name: newTemplateName.value.trim(),
    roles: JSON.parse(JSON.stringify(roles.value)),
    shifts: JSON.parse(JSON.stringify(shifts.value))
  })
  saveTemplatesToStorage()
  newTemplateName.value = ''
}
const loadTemplate = (template) => {
  if (!template) return
  roles.value = JSON.parse(JSON.stringify(template.roles))
  shifts.value = JSON.parse(JSON.stringify(template.shifts))
}

onMounted(() => loadTemplatesFromStorage())

const addShift = () => shifts.value.push({ id: createId(), name: '', start: '', end: '', break: 0 })
const removeShift = index => shifts.value.splice(index, 1)
const addRole = () => roles.value.push({ id: createId(), name: '', wte: 1, visitTypes: [] })
const removeRole = index => roles.value.splice(index, 1)
const addVisitType = role => role.visitTypes.push({ id: createId(), name: '', length: 0, count: 0, shiftIndex: 0 })
const removeVisitType = (role, idx) => role.visitTypes.splice(idx, 1)

const getMinutesBetweenTimes = (start, end, breakMins) => {
  const [sh, sm] = start.split(':').map(Number)
  const [eh, em] = end.split(':').map(Number)
  let duration = (eh * 60 + em) - (sh * 60 + sm) - breakMins
  return duration < 0 ? duration + 1440 : duration
}

const validateInputs = () => {
  if (!shifts.value.length) return 'Please configure at least one shift.'
  if (!roles.value.length) return 'Please configure at least one role.'
  for (const r of roles.value) if (!r.visitTypes.length) return `Add visit types for ${r.name}`
  return null
}

const calculateDemand = () => {
  warningMessage.value = ''
  const warn = validateInputs()
  if (warn) return warningMessage.value = warn

  demandOutput.value = roles.value.map(role => {
    const totalVisits = role.visitTypes.reduce((s, vt) => s + vt.count, 0)
    const totalVisitTime = role.visitTypes.reduce((s, vt) => s + vt.length * vt.count, 0)
    const totalTravelTime = role.visitTypes.reduce((s, vt) => s + averageTravelTime.value * vt.count, 0)
    const totalMinutes = totalVisitTime + totalTravelTime
    const uniqueIndices = [...new Set(role.visitTypes.map(vt => vt.shiftIndex))]
    const averageShiftTime = uniqueIndices.length
      ? uniqueIndices.reduce((sum, i) => sum + getMinutesBetweenTimes(shifts.value[i].start, shifts.value[i].end, shifts.value[i].break), 0) / uniqueIndices.length
      : 1
    const requiredShifts = Math.ceil(totalMinutes / averageShiftTime)
    const totalWeeklyHours = (totalMinutes * 7) / 60
    const wteRequired = (totalWeeklyHours / 37.5).toFixed(2)
    const wteComparison = parseFloat(wteRequired) - (role.wte || 0)
    const wteStatus = wteComparison > 0 ? '⚠️ Over budget' : wteComparison < 0 ? '✅ Under budget' : '✅ Matches budget'

    const shiftCounts = {}
    role.visitTypes.forEach(vt => {
      const name = shifts.value[vt.shiftIndex]?.name || 'Unknown'
      const timePerVisit = vt.length + averageTravelTime.value
      const totalTimeForType = timePerVisit * vt.count
      const availableTime = getMinutesBetweenTimes(shifts.value[vt.shiftIndex].start, shifts.value[vt.shiftIndex].end, shifts.value[vt.shiftIndex].break) || 1
      shiftCounts[name] = (shiftCounts[name] || 0) + Math.ceil(totalTimeForType / availableTime)
    })
    const totalShifts = Object.values(shiftCounts).reduce((sum, c) => sum + c, 0)
    const perShiftBreakdown = role.visitTypes.reduce((obj, vt) => {
      const nm = shifts.value[vt.shiftIndex]?.name || 'Unknown'
      if (!obj[nm]) obj[nm] = { visits: 0, visitTime: 0, travelTime: 0, totalTime: 0 }
      obj[nm].visits += vt.count
      obj[nm].visitTime += vt.length * vt.count
      obj[nm].travelTime += averageTravelTime.value * vt.count
      obj[nm].totalTime += vt.length * vt.count + averageTravelTime.value * vt.count
      return obj
    }, {})
    return { name: role.name, totalVisits, totalVisitTimeMins: totalVisitTime, totalTravelTimeMins: totalTravelTime, totalMinutes, totalHours: (totalMinutes/60).toFixed(2), shiftMinutes: averageShiftTime, shiftHours: (averageShiftTime/60).toFixed(2), requiredShifts, perShiftBreakdown, shiftCounts, totalShifts, wteRequired, wteStatus }
  })
}

const exportCSV = () => {
  if (!demandOutput.value.length) return
  const rows = []
  rows.push(['Role','Total Visits','Total Time (hrs)','Total Shifts','WTE Required','Shift Breakdown'])
  demandOutput.value.forEach(role => {
    const breakdown = Object.entries(role.shiftCounts).map(([s,c])=>`${s}:${c}`).join('; ')
    rows.push([role.name, role.totalVisits, role.totalHours, role.totalShifts, role.wteRequired, breakdown])
  })
  const csv = rows.map(r=>r.map(v=>`"${v}"`).join(',')).join('\n')
  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a'); link.href = URL.createObjectURL(blob); link.setAttribute('download','demand_summary.csv'); document.body.appendChild(link); link.click(); document.body.removeChild(link)
}

const exportPDF = () => {
  const doc = new jsPDF()
  doc.text('Demand Summary',14,20)
  let y=30
  demandOutput.value.forEach(r=>{ doc.text(`Role: ${r.name} - Visits: ${r.totalVisits}, Time: ${r.totalHours}hrs, Shifts: ${r.totalShifts}`,14,y); y+=10 })
  doc.save('demand_summary.pdf')
}

watch([
  roles,
  shifts,
  averageTravelTime
], debounce(() => { if (roles.value.length && shifts.value.length) calculateDemand() }, 300), { deep: true })
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined');

.field {
  background-color: #fffbeb; /* light yellow background */
  border: 2px dashed #f59e0b; /* dashed orange border */
  color: #1f2937; /* dark text for contrast */
  padding: 0.75rem;
  border-radius: 0.5rem;
  width: 100%;
  transition: background-color 0.2s ease-in-out, border-color 0.2s ease-in-out;
}
.field:focus {
  outline: none;
  background-color: #fef3c7; /* slightly darker yellow on focus */
  border-color: #d97706; /* darker orange border on focus */
  box-shadow: 0 0 0 4px rgba(244, 143, 0, 0.3);
}
.field:focus {
  outline: none;
  border-color: #2563eb; /* deeper blue on focus */
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.4);
}

.action {
  background-color: #2563eb;
  color: white;
  padding: 0.5rem 1rem;
  border: 2px solid #2563eb;
  border-radius: 0.375rem;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  transition: transform 0.2s ease-in-out, background-color 0.2s;
  cursor: pointer;
}
.action:hover {
  background-color: #1e40af;
  transform: scale(1.02);
}

.fade-enter-active, .fade-leave-active {
  transition: all 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: scale(0.95);
}
</style>
