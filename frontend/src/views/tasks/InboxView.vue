<template>
	<div class="inbox-page">

		<!-- Toolbar: Tabs & Controls -->
		<div class="toolbar">
			<div class="tabs-container">
				<button
					type="button"
					class="tab-btn"
					:class="{active: activeTab === 'all'}"
					@click="activeTab = 'all'"
				>
					Todas
				</button>
				<button
					type="button"
					class="tab-btn"
					:class="{active: activeTab === 'assigned'}"
					@click="activeTab = 'assigned'"
				>
					Atribuídas a mim
				</button>
			</div>
			<div class="toolbar-controls">
				<div class="dropdown-wrapper">
					<button
						type="button"
						class="control-btn"
						@click.stop="toggleSortMenu"
					>
						<Icon icon="sort" />
						<span>{{ currentSortLabel }}</span>
					</button>
					<div
						v-if="showSortMenu"
						class="dropdown-menu"
					>
						<div
							class="dropdown-item"
							@click="setSort('date')"
						>
							Data de entrega
						</div>
						<div
							class="dropdown-item"
							@click="setSort('priority')"
						>
							Prioridade
						</div>
						<div
							class="dropdown-item"
							@click="setSort('title')"
						>
							Nome da tarefa
						</div>
					</div>
				</div>

				<div class="dropdown-wrapper">
					<button
						type="button"
						class="control-btn"
						@click.stop="toggleFilterMenu"
					>
						<Icon icon="filter" />
						<span>{{ currentFilterLabel }}</span>
					</button>
					<div
						v-if="showFilterMenu"
						class="dropdown-menu"
					>
						<div
							class="dropdown-item"
							@click="setFilter('all')"
						>
							Todas as tarefas
						</div>
						<div
							class="dropdown-item"
							@click="setFilter('pending')"
						>
							Apenas pendentes
						</div>
						<div
							class="dropdown-item"
							@click="setFilter('completed')"
						>
							Apenas concluídas
						</div>
					</div>
				</div>
			</div>
		</div>

		<!-- Main Content Area -->
		<div class="content-area">
			<div class="table-container">
				<!-- Task Creation Input Row -->
				<div class="task-input-row">
					<span class="task-input-icon">
						<Icon icon="list-ul" />
					</span>
					<input
						v-model="newTaskTitle"
						type="text"
						class="task-input"
						placeholder="Adicionar uma tarefa..."
						@keyup.enter="handleCreateTask"
					>
					<button
						type="button"
						class="add-btn"
						:disabled="!newTaskTitle.trim()"
						@click="handleCreateTask"
					>
						Add
					</button>
				</div>

				<!-- Table Header Row -->
				<div class="table-header-row">
					<div class="th-col col-check" />
					<div class="th-col col-task">
						Tarefa
					</div>
					<div class="th-col col-project">
						Projeto
					</div>
					<div class="th-col col-date">
						Data de entrega
					</div>
					<div class="th-col col-assignee">
						Responsável
					</div>
					<div class="th-col col-subtasks">
						Subtarefas
					</div>
					<div class="th-col col-actions" />
				</div>

				<!-- Task List Rows -->
				<div
					v-if="filteredTasks.length > 0"
					class="table-body"
				>
					<div
						v-for="task in filteredTasks"
						:key="task.id"
						class="grid-row"
						:class="{completed: task.done}"
						@click="openTaskDetail(task)"
					>
						<!-- Checkbox -->
						<div
							class="cell col-check"
							@click.stop
						>
							<input
								type="checkbox"
								class="task-checkbox"
								:checked="task.done"
								:aria-label="`Concluir tarefa: ${task.title}`"
								@change="toggleTaskDone(task)"
							>
						</div>

						<!-- Task Title & Labels -->
						<div class="cell col-task">
							<span
								class="priority-icon"
								:class="`priority-${getPriorityClass(task.priority)}`"
								:title="`Prioridade ${task.priority}`"
							>
								<Icon icon="bolt" />
							</span>
							<span class="task-title">{{ task.title }}</span>
							<div
								v-if="task.labels && task.labels.length > 0"
								class="labels-container"
							>
								<span
									v-for="lbl in task.labels"
									:key="lbl.id"
									class="label-pill"
									:style="{borderColor: lbl.hex_color ? `#${lbl.hex_color}` : undefined}"
								>
									{{ lbl.title }}
								</span>
							</div>
						</div>

						<!-- Project -->
						<div class="cell col-project">
							<div
								v-if="task.project"
								class="project-pill"
							>
								<span
									class="project-dot"
									:style="{backgroundColor: getProjectColor(task.project)}"
								/>
								<span>{{ task.project.title || 'Inbox' }}</span>
							</div>
						</div>

						<!-- Due Date -->
						<div class="cell col-date">
							<span>{{ formatDueDate(task.due_date) }}</span>
						</div>

						<!-- Assignee -->
						<div class="cell col-assignee">
							<div
								v-if="task.assignees && task.assignees.length > 0"
								class="avatar-mini"
								:title="task.assignees[0].name || task.assignees[0].username"
							>
								{{ getInitials(task.assignees[0].name || task.assignees[0].username) }}
							</div>
							<span
								v-else
								class="text-muted-dash"
							>-</span>
						</div>

						<!-- Subtasks -->
						<div class="cell col-subtasks">
							<div
								v-if="task.subtask_count"
								class="subtask-indicator"
							>
								<Icon icon="list-check" />
								<span>{{ task.subtask_done_count || 0 }}/{{ task.subtask_count }}</span>
							</div>
							<span
								v-else
								class="text-muted-dash"
							>-</span>
						</div>

						<!-- Row Actions (•••) -->
						<div
							class="cell col-actions"
							@click.stop
						>
							<div class="row-actions">
								<button
									type="button"
									class="action-btn"
									title="Ações"
									@click.stop="toggleRowMenu(task.id)"
								>
									•••
								</button>
								<div
									v-if="activeRowMenuId === task.id"
									class="dropdown-menu row-menu"
								>
									<div
										class="dropdown-item"
										@click="toggleTaskDone(task); activeRowMenuId = null"
									>
										{{ task.done ? 'Marcar como Pendente' : 'Marcar como Concluída' }}
									</div>
									<div
										class="dropdown-item danger"
										@click="deleteTask(task); activeRowMenuId = null"
									>
										Excluir
									</div>
								</div>
							</div>
						</div>
					</div>
				</div>

				<!-- Empty State -->
				<div
					v-else
					class="empty-view"
				>
					<div class="empty-icon">
						<svg
							width="36"
							height="36"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="1.8"
							stroke-linecap="round"
							stroke-linejoin="round"
						>
							<path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z" />
							<path d="m9 12 2 2 4-4" />
						</svg>
					</div>
					<h3 class="empty-title">
						Tudo limpo por aqui!
					</h3>
					<p class="empty-text">
						Nenhuma tarefa pendente na caixa de entrada.
						<a
							class="empty-link"
							@click="focusInput"
						>Criar uma tarefa.</a>
					</p>
				</div>
			</div>
		</div>

		<!-- Toast Notification -->
		<div
			v-if="toastMessage"
			class="toast-notification"
		>
			{{ toastMessage }}
		</div>
	</div>
</template>

<script setup lang="ts">
import {computed, onMounted, onUnmounted, ref} from 'vue'
import {useRouter} from 'vue-router'
import {useAuthStore} from '@/stores/auth'
import Icon from '@/components/misc/Icon'

interface TaskItem {
	id: number
	title: string
	done: boolean
	priority: number
	due_date?: string | null
	project?: {id: number; title: string; hex_color?: string}
	labels?: Array<{id: number; title: string; hex_color?: string}>
	assignees?: Array<{id: number; username: string; name?: string}>
	subtask_count?: number
	subtask_done_count?: number
}

const router = useRouter()
const authStore = useAuthStore()

const userInitials = computed(() => {
	const name = authStore.userDisplayName || 'Matheus'
	return name.substring(0, 2).toUpperCase()
})

const activeTab = ref<'all' | 'assigned'>('all')
const newTaskTitle = ref('')
const activeSort = ref<'date' | 'priority' | 'title'>('date')
const activeFilter = ref<'all' | 'pending' | 'completed'>('all')

const showSortMenu = ref(false)
const showFilterMenu = ref(false)
const activeRowMenuId = ref<number | null>(null)
const toastMessage = ref('')
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(msg: string) {
	toastMessage.value = msg
	if (toastTimer) clearTimeout(toastTimer)
	toastTimer = setTimeout(() => {
		toastMessage.value = ''
	}, 3000)
}

// In-memory tasks state aligned with Vikunja design proposal
const tasks = ref<TaskItem[]>([
	{
		id: 101,
		title: 'Validar tokens do design system Linear Style',
		done: false,
		priority: 3,
		due_date: '2026-11-08',
		project: {id: 1, title: 'mimirManager', hex_color: '5e6ad2'},
		labels: [{id: 1, title: 'UI', hex_color: '8b5cf6'}],
		assignees: [{id: 1, username: 'Matheus', name: 'Matheus Oliveira'}],
		subtask_count: 3,
		subtask_done_count: 1,
	},
	{
		id: 102,
		title: 'Implementar tela de Inbox conforme spec-inbox-deepseek.md',
		done: false,
		priority: 2,
		due_date: '2026-11-09',
		project: {id: 1, title: 'mimirManager', hex_color: '5e6ad2'},
		labels: [{id: 2, title: 'Core', hex_color: 'e5484d'}],
		assignees: [{id: 1, username: 'Matheus', name: 'Matheus Oliveira'}],
		subtask_count: 4,
		subtask_done_count: 2,
	},
	{
		id: 103,
		title: 'Revisar fluxo de autenticação e rotas protegidas',
		done: true,
		priority: 1,
		due_date: '2026-11-07',
		project: {id: 2, title: 'Frontend', hex_color: '10b981'},
		labels: [{id: 3, title: 'Auth', hex_color: 'f5a623'}],
		assignees: [{id: 1, username: 'Matheus', name: 'Matheus Oliveira'}],
		subtask_count: 2,
		subtask_done_count: 2,
	},
])

const currentSortLabel = computed(() => {
	switch (activeSort.value) {
		case 'priority': return 'Ordenar: Prioridade'
		case 'title': return 'Ordenar: Nome'
		default: return 'Ordenar: Data'
	}
})

const currentFilterLabel = computed(() => {
	switch (activeFilter.value) {
		case 'pending': return 'Filtro: Pendentes'
		case 'completed': return 'Filtro: Concluídas'
		default: return 'Filtro: Todas'
	}
})

const filteredTasks = computed(() => {
	let list = [...tasks.value]

	if (activeFilter.value === 'pending') {
		list = list.filter(t => !t.done)
	} else if (activeFilter.value === 'completed') {
		list = list.filter(t => t.done)
	}

	if (activeSort.value === 'priority') {
		list.sort((a, b) => b.priority - a.priority)
	} else if (activeSort.value === 'title') {
		list.sort((a, b) => a.title.localeCompare(b.title))
	} else if (activeSort.value === 'date') {
		list.sort((a, b) => String(a.due_date || '').localeCompare(String(b.due_date || '')))
	}

	return list
})

function toggleSortMenu() {
	showSortMenu.value = !showSortMenu.value
	showFilterMenu.value = false
}

function toggleFilterMenu() {
	showFilterMenu.value = !showFilterMenu.value
	showSortMenu.value = false
}

function setSort(sort: 'date' | 'priority' | 'title') {
	activeSort.value = sort
	showSortMenu.value = false
	showToast(`Ordenação atualizada para: ${currentSortLabel.value}`)
}

function setFilter(filter: 'all' | 'pending' | 'completed') {
	activeFilter.value = filter
	showFilterMenu.value = false
	showToast(`Filtro aplicado: ${currentFilterLabel.value}`)
}

function toggleRowMenu(id: number) {
	activeRowMenuId.value = activeRowMenuId.value === id ? null : id
}

function toggleTaskDone(task: TaskItem) {
	task.done = !task.done
	showToast(task.done ? 'Tarefa marcada como concluída!' : 'Tarefa desmarcada.')
}

function handleCreateTask() {
	if (!newTaskTitle.value.trim()) return
	const newTask: TaskItem = {
		id: Date.now(),
		title: newTaskTitle.value.trim(),
		done: false,
		priority: 2,
		due_date: 'Hoje',
		project: {id: 1, title: 'mimirManager', hex_color: '5e6ad2'},
		labels: [{id: 4, title: 'Nova', hex_color: '5e6ad2'}],
		assignees: [{id: 1, username: 'Matheus', name: 'Matheus Oliveira'}],
		subtask_count: 0,
		subtask_done_count: 0,
	}
	tasks.value.unshift(newTask)
	newTaskTitle.value = ''
	showToast('Tarefa criada com sucesso!')
}

function deleteTask(task: TaskItem) {
	tasks.value = tasks.value.filter(t => t.id !== task.id)
	showToast('Tarefa excluída.')
}

function openTaskDetail(task: TaskItem) {
	showToast(`Abrindo detalhes da tarefa #${task.id}...`)
}

function focusInput() {
	const el = document.querySelector('.task-input') as HTMLInputElement | null
	el?.focus()
}

function getPriorityClass(priority: number) {
	if (priority >= 4) return 'urgent'
	if (priority === 3) return 'high'
	if (priority === 2) return 'medium'
	return 'low'
}

function getProjectColor(project?: {hex_color?: string}) {
	return project?.hex_color ? `#${project.hex_color}` : '#5e6ad2'
}

function formatDueDate(date?: string | null) {
	if (!date) return '-'
	return date
}

function getInitials(name: string) {
	return name.substring(0, 2).toUpperCase()
}

function handleClickOutside() {
	showSortMenu.value = false
	showFilterMenu.value = false
	activeRowMenuId.value = null
}

onMounted(() => {
	window.addEventListener('click', handleClickOutside)
})

onUnmounted(() => {
	window.removeEventListener('click', handleClickOutside)
	if (toastTimer) clearTimeout(toastTimer)
})
</script>

<style lang="scss" scoped>
.inbox-page {
	/* Design Tokens */
	--font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
	--bg-app: #f9f9f9;
	--bg-sidebar: #fcfcfc;
	--bg-card: #ffffff;
	--bg-hover: #f4f4f5;
	--bg-header-row: #fafafa;
	--bg-modal: #ffffff;
	--text-primary: #111111;
	--text-secondary: #6b6b6b;
	--text-tertiary: #9ca3af;
	--border-color: #e5e5e5;
	--border-subtle: #f0f0f0;
	--border-focus: #5e6ad2;
	--accent-base: #5e6ad2;
	--accent-hover: #4a55b5;
	--priority-urgent: #e5484d;
	--priority-high: #f5a623;
	--priority-medium: #5e6ad2;
	--priority-low: #9ca3af;
	--shadow-dropdown: 0 4px 12px rgba(0, 0, 0, 0.08);

	font-family: var(--font-family);
	background-color: var(--bg-app);
	color: var(--text-primary);
	min-height: 100vh;
	display: flex;
	flex-direction: column;
	width: 100%;
}

:global(html.dark) .inbox-page {
	--bg-app: #0f0f0f;
	--bg-sidebar: #0f0f0f;
	--bg-card: #1c1c1c;
	--bg-hover: #262a36;
	--bg-header-row: #181b22;
	--bg-modal: #1c1c1c;
	--text-primary: #f0f2f5;
	--text-secondary: #9aa1b2;
	--text-tertiary: #656c80;
	--border-color: #262a36;
	--border-subtle: #1e222c;
}

.top-header {
	height: 48px;
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 0 24px;
	border-bottom: 1px solid var(--border-color);
	background: var(--bg-app);
}

.breadcrumb {
	display: flex;
	align-items: center;
	gap: 8px;
	font-size: 14px;
	font-weight: 600;
	color: var(--text-primary);

	.breadcrumb-item {
		color: var(--text-secondary);
	}

	.breadcrumb-separator {
		color: var(--text-tertiary);
	}
}

.header-actions {
	display: flex;
	align-items: center;
	gap: 12px;
}

.icon-btn {
	background: transparent;
	border: none;
	cursor: pointer;
	color: var(--text-secondary);
	display: flex;
	padding: 6px;
	border-radius: 4px;

	&:hover {
		background: var(--bg-hover);
		color: var(--text-primary);
	}
}

.user-profile {
	display: flex;
	align-items: center;
	gap: 8px;
	font-size: 13px;
	font-weight: 500;
	cursor: pointer;
}

.avatar {
	width: 26px;
	height: 26px;
	border-radius: 50%;
	background: #e0e7ff;
	color: var(--accent-base);
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 11px;
	font-weight: 600;
}

.username {
	color: var(--text-primary);
}

.toolbar {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 16px 24px;
}

.tabs-container {
	display: flex;
	background: #f0f0f0;
	padding: 3px;
	border-radius: 6px;
}

:global(html.dark) .tabs-container {
	background: #262a36;
}

.tab-btn {
	background: transparent;
	border: none;
	padding: 5px 14px;
	font-size: 13px;
	font-weight: 500;
	color: var(--text-secondary);
	border-radius: 4px;
	cursor: pointer;

	&.active {
		background: var(--bg-card);
		color: var(--text-primary);
		box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
	}
}

.toolbar-controls {
	display: flex;
	gap: 8px;
}

.dropdown-wrapper {
	position: relative;
}

.control-btn {
	display: flex;
	align-items: center;
	gap: 6px;
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 6px;
	padding: 6px 12px;
	font-size: 12px;
	font-weight: 500;
	color: var(--text-secondary);
	cursor: pointer;

	&:hover {
		background: var(--bg-hover);
		color: var(--text-primary);
	}
}

.dropdown-menu {
	position: absolute;
	top: 100%;
	right: 0;
	margin-top: 4px;
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 6px;
	box-shadow: var(--shadow-dropdown);
	min-width: 160px;
	z-index: 100;
	padding: 4px;
}

.dropdown-item {
	padding: 8px 10px;
	font-size: 13px;
	color: var(--text-primary);
	border-radius: 4px;
	cursor: pointer;
	display: flex;
	align-items: center;
	gap: 8px;

	&:hover {
		background: var(--bg-hover);
	}

	&.danger {
		color: var(--priority-urgent);
	}
}

.content-area {
	flex: 1;
	padding: 0 24px 24px 24px;
	overflow-y: auto;
}

.table-container {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 8px;
	box-shadow: 0 1px 3px rgba(0, 0, 0, 0.02);
	display: flex;
	flex-direction: column;
	min-width: 800px;
}

.task-input-row {
	display: flex;
	align-items: center;
	padding: 10px 18px;
	border-bottom: 1px solid var(--border-color);
	gap: 12px;

	&:focus-within {
		background: var(--bg-header-row);
	}
}

.task-input-icon {
	color: var(--text-tertiary);
	display: flex;
	font-size: 14px;
}

.task-input {
	flex: 1;
	border: none;
	background: transparent;
	font-size: 14px;
	color: var(--text-primary);
	outline: none;
	font-family: var(--font-family);

	&::placeholder {
		color: var(--text-tertiary);
	}
}

.add-btn {
	background: var(--accent-base);
	color: white;
	border: none;
	border-radius: 6px;
	padding: 5px 14px;
	font-size: 12px;
	font-weight: 500;
	cursor: pointer;
	opacity: 0;
	transition: opacity 0.15s ease;

	.task-input-row:focus-within & {
		opacity: 1;
	}

	&:disabled {
		opacity: 0.5 !important;
		cursor: not-allowed;
	}
}

.table-header-row,
.grid-row {
	display: grid;
	/* Grid com gap: 16px exato corrigindo espaço entre responsável e subtarefas */
	grid-template-columns: 32px minmax(300px, 1fr) 140px 100px 100px 80px 40px;
	gap: 16px;
	align-items: center;
	padding: 10px 18px;
}

.table-header-row {
	background: var(--bg-header-row);
	border-bottom: 1px solid var(--border-color);
	font-size: 11px;
	font-weight: 600;
	color: var(--text-secondary);
	text-transform: uppercase;
	letter-spacing: 0.5px;
	border-top-left-radius: 8px;
	border-top-right-radius: 8px;
}

.grid-row {
	border-bottom: 1px solid var(--border-subtle);
	transition: background 0.1s ease;
	cursor: pointer;
	position: relative;

	&:hover {
		background: var(--bg-hover);
	}

	&.completed {
		opacity: 0.5;

		.task-title {
			text-decoration: line-through;
		}
	}
}

.col-check {
	display: flex;
	align-items: center;
}

.col-task {
	display: flex;
	align-items: center;
	gap: 8px;
	min-width: 0;
}

.col-project {
	display: flex;
	align-items: center;
}

.col-date {
	display: flex;
	align-items: center;
	font-size: 12px;
	color: var(--text-secondary);
}

.col-assignee {
	display: flex;
	align-items: center;
}

.col-subtasks {
	display: flex;
	align-items: center;
	font-size: 12px;
	color: var(--text-secondary);
}

.col-actions {
	display: flex;
	justify-content: flex-end;
	position: relative;
}

.task-checkbox {
	appearance: none;
	width: 16px;
	height: 16px;
	border: 1.5px solid var(--text-tertiary);
	border-radius: 4px;
	cursor: pointer;
	position: relative;
	transition: all 0.15s ease;

	&:hover {
		border-color: var(--accent-base);
	}

	&:checked {
		background: var(--accent-base);
		border-color: var(--accent-base);

		&::after {
			content: '';
			position: absolute;
			top: 2px;
			left: 4px;
			width: 4px;
			height: 8px;
			border: solid white;
			border-width: 0 2px 2px 0;
			transform: rotate(45deg);
		}
	}
}

.priority-icon {
	display: inline-flex;
	align-items: center;
	justify-content: center;
	font-size: 12px;
	flex-shrink: 0;

	&.priority-urgent {
		color: var(--priority-urgent);
	}
	&.priority-high {
		color: var(--priority-high);
	}
	&.priority-medium {
		color: var(--priority-medium);
	}
	&.priority-low {
		color: var(--priority-low);
	}
}

.task-title {
	font-size: 13px;
	font-weight: 500;
	color: var(--text-primary);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.labels-container {
	display: flex;
	gap: 4px;
	flex-shrink: 0;
}

.label-pill {
	font-size: 10px;
	font-weight: 600;
	padding: 2px 6px;
	border-radius: 4px;
	border: 1px solid var(--border-color);
	background: var(--bg-app);
	color: var(--text-secondary);
}

.project-pill {
	font-size: 11px;
	font-weight: 500;
	padding: 2px 8px;
	border-radius: 12px;
	background: #f0f0f0;
	color: var(--text-secondary);
	display: flex;
	align-items: center;
	gap: 6px;
}

:global(html.dark) .project-pill {
	background: #262a36;
}

.project-dot {
	width: 6px;
	height: 6px;
	border-radius: 50%;
}

.avatar-mini {
	width: 22px;
	height: 22px;
	border-radius: 50%;
	background: var(--accent-base);
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 9px;
	font-weight: 600;
	color: white;
}

.text-muted-dash {
	color: var(--text-tertiary);
	font-size: 13px;
}

.subtask-indicator {
	display: flex;
	align-items: center;
	gap: 5px;
	font-size: 12px;
	color: var(--text-secondary);
}

.row-actions {
	opacity: 0;
	transition: opacity 0.15s ease;
	background: inherit;

	.grid-row:hover & {
		opacity: 1;
	}
}

.action-btn {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 4px;
	padding: 3px 8px;
	cursor: pointer;
	color: var(--text-secondary);
	font-size: 11px;
	display: flex;
	align-items: center;

	&:hover {
		background: var(--bg-hover);
		color: var(--text-primary);
	}
}

.row-menu {
	right: 0;
	top: 100%;
}

.empty-view {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 64px 24px;
	text-align: center;
}

.empty-icon {
	width: 80px;
	height: 80px;
	background: var(--bg-hover);
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	margin-bottom: 16px;
	color: var(--accent-base);
}

.empty-title {
	font-size: 16px;
	font-weight: 600;
	color: var(--text-primary);
	margin-bottom: 8px;
}

.empty-text {
	font-size: 14px;
	color: var(--text-secondary);
}

.empty-link {
	color: var(--accent-base);
	text-decoration: none;
	font-weight: 500;
	margin-left: 4px;
	cursor: pointer;

	&:hover {
		text-decoration: underline;
	}
}

.toast-notification {
	position: fixed;
	bottom: 24px;
	left: 50%;
	transform: translateX(-50%);
	background: #111111;
	color: #ffffff;
	padding: 10px 20px;
	border-radius: 8px;
	font-size: 13px;
	font-weight: 500;
	box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
	z-index: 2000;
	pointer-events: none;
	animation: toastFade 0.2s ease;
}

@keyframes toastFade {
	from {
		opacity: 0;
		transform: translate(-50%, 10px);
	}
	to {
		opacity: 1;
		transform: translate(-50%, 0);
	}
}
</style>
