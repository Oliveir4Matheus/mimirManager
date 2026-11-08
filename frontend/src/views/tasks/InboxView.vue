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
							Todas
						</div>
						<div
							class="dropdown-item"
							@click="setFilter('pending')"
						>
							Pendentes
						</div>
						<div
							class="dropdown-item"
							@click="setFilter('completed')"
						>
							Concluídas
						</div>
					</div>
				</div>
			</div>
		</div>

		<!-- Main Card: Input + Table -->
		<div class="main-card-wrapper">
			<div class="tasks-card">
				<!-- Input Container -->
				<div class="task-input-container">
					<div class="input-icon">
						<svg
							width="16"
							height="16"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
						>
							<line x1="8" y1="6" x2="21" y2="6" />
							<line x1="8" y1="12" x2="21" y2="12" />
							<line x1="8" y1="18" x2="21" y2="18" />
							<line x1="3" y1="6" x2="3.01" y2="6" />
							<line x1="3" y1="12" x2="3.01" y2="12" />
							<line x1="3" y1="18" x2="3.01" y2="18" />
						</svg>
					</div>
					<input
						ref="taskInputRef"
						v-model="newTaskTitle"
						type="text"
						class="task-input"
						placeholder="Add a task..."
						aria-label="Adicionar uma tarefa"
						@keydown.enter="createTask"
					>
					<button
						type="button"
						class="add-btn"
						aria-label="Adicionar tarefa"
						:disabled="createTaskMutation.isPending.value"
						@click="createTask"
					>
						Add
					</button>
				</div>

				<!-- Table List (Populated) -->
				<div
					v-if="filteredTasks.length > 0"
					class="table-container"
					role="grid"
					aria-label="Lista de Tarefas da Caixa de Entrada"
				>
					<!-- Header Row -->
					<div
						class="grid-row table-header-row"
						role="row"
					>
						<div class="cell col-check" />
						<div class="cell col-title">
							TAREFA
						</div>
						<div class="cell col-project">
							PROJETO
						</div>
						<div class="cell col-date">
							DATA DE ENTREGA
						</div>
						<div class="cell col-assignee">
							RESPONSÁVEL
						</div>
						<div class="cell col-subtasks">
							SUBTAREFAS
						</div>
						<div class="cell col-actions" />
					</div>

					<!-- Task Rows -->
					<div
						v-for="task in filteredTasks"
						:key="task.id"
						class="grid-row task-row"
						:class="{completed: task.done}"
						role="row"
						@click="openTaskDetails(task)"
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
								:aria-label="'Concluir tarefa: ' + task.title"
								@change="toggleTaskDone(task)"
							>
						</div>

						<!-- Task Title & Badges -->
						<div class="cell col-title">
							<span
								class="priority-icon"
								:class="'priority-' + getPriorityName(task.priority)"
								:title="'Prioridade: ' + getPriorityName(task.priority)"
							>
								<!-- Urgent: Triangle -->
								<svg
									v-if="task.priority === 5 || task.priority === 4"
									viewBox="0 0 24 24"
									fill="currentColor"
								>
									<path d="M12 2L2 22h20L12 2z" fill="none" stroke="currentColor" stroke-width="2" />
									<line x1="12" y1="9" x2="12" y2="15" stroke="currentColor" stroke-width="2" stroke-linecap="round" />
									<circle cx="12" cy="18" r="1" fill="currentColor" />
								</svg>
								<!-- High: Tall double bars -->
								<svg
									v-else-if="task.priority === 3"
									viewBox="0 0 24 24"
									fill="currentColor"
								>
									<rect x="6" y="10" width="4" height="10" rx="1" />
									<rect x="14" y="4" width="4" height="16" rx="1" />
								</svg>
								<!-- Medium: Plus / Bar -->
								<svg
									v-else-if="task.priority === 2"
									viewBox="0 0 24 24"
									fill="currentColor"
								>
									<path d="M12 4v16m-8-8h16" stroke="currentColor" stroke-width="2" stroke-linecap="round" />
								</svg>
								<!-- Low: Small bars -->
								<svg
									v-else
									viewBox="0 0 24 24"
									fill="currentColor"
								>
									<rect x="6" y="14" width="4" height="6" rx="1" />
									<rect x="14" y="14" width="4" height="6" rx="1" />
								</svg>
							</span>

							<span class="task-title-text">{{ task.title }}</span>

							<div
								v-if="task.labels && task.labels.length > 0"
								class="labels-group"
							>
								<span
									v-for="lbl in task.labels"
									:key="lbl.id"
									class="label-badge"
									:style="{color: '#' + (lbl.hex_color || '5e6ad2'), backgroundColor: '#' + (lbl.hex_color || '5e6ad2') + '1a'}"
								>
									{{ lbl.title }}
								</span>
							</div>
						</div>

						<!-- Project Pill -->
						<div class="cell col-project">
							<div class="project-pill">
								<span
									class="project-dot"
									:style="{backgroundColor: getProjectColor(task.project_id)}"
								/>
								<span>{{ getProjectName(task.project_id) }}</span>
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
								v-if="getSubtaskCount(task) > 0"
								class="subtask-indicator"
							>
								<Icon icon="list-check" />
								<span>{{ getSubtaskDoneCount(task) }}/{{ getSubtaskCount(task) }}</span>
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
					v-else-if="!taskQuery.isFetching.value"
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
import {useProjects} from '@/composables/useProjects'
import {useTasks} from '@/composables/useTasks'
import {useCreateTaskMutation, useUpdateTaskMutation, useDeleteTaskMutation} from '@/client/queries/taskMutations'
import Icon from '@/components/misc/Icon'
import {getProjectTitle} from '@/helpers/getProjectTitle'
import type {TaskResponse, TaskScope} from '@/client/queries/tasks'

const router = useRouter()
const authStore = useAuthStore()
const projectList = useProjects()

const createTaskMutation = useCreateTaskMutation()
const updateTaskMutation = useUpdateTaskMutation(true)
const deleteTaskMutation = useDeleteTaskMutation()

const activeTab = ref<'all' | 'assigned'>('all')
const newTaskTitle = ref('')
const activeSort = ref<'date' | 'priority' | 'title'>('date')
const activeFilter = ref<'all' | 'pending' | 'completed'>('all')

const showSortMenu = ref(false)
const showFilterMenu = ref(false)
const activeRowMenuId = ref<number | null>(null)
const taskInputRef = ref<HTMLInputElement | null>(null)

const toastMessage = ref('')
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(msg: string) {
	toastMessage.value = msg
	if (toastTimer) clearTimeout(toastTimer)
	toastTimer = setTimeout(() => {
		toastMessage.value = ''
	}, 3000)
}

// Target Inbox project ID
const inboxProjectId = computed(() => {
	return authStore.settings?.default_project_id || 
		projectList.projectsArray[0]?.id || 
		1
})

const taskScope = computed<TaskScope>(() => ({
	project: inboxProjectId.value,
	params: {
		sort_by: activeSort.value === 'title' ? ['title', 'id'] : (activeSort.value === 'priority' ? ['priority', 'id'] : ['due_date', 'id']),
		order_by: activeSort.value === 'priority' ? ['desc', 'desc'] : ['asc', 'desc'],
		expand: ['subtasks'],
	},
}))

const taskQuery = useTasks(taskScope, {
	enabled: () => authStore.authenticated && Boolean(inboxProjectId.value),
})

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
	let list = [...(taskQuery.tasks.value ?? [])]

	// Tab: Assigned to me
	if (activeTab.value === 'assigned') {
		const currentUserId = authStore.info?.id
		list = list.filter(t => t.assignees?.some(a => a.id === currentUserId))
	}

	// Filter: Pending / Completed
	if (activeFilter.value === 'pending') {
		list = list.filter(t => !t.done)
	} else if (activeFilter.value === 'completed') {
		list = list.filter(t => t.done)
	}

	return list
})

function getProjectName(projectId: number): string {
	const p = projectList.projects[projectId]
	return p ? getProjectTitle(p) : 'Inbox'
}

function getProjectColor(projectId: number): string {
	const p = projectList.projects[projectId]
	return p?.hex_color ? `#${p.hex_color}` : '#9ca3af'
}

function getPriorityName(priority: number): 'urgent' | 'high' | 'medium' | 'low' {
	if (priority >= 4) return 'urgent'
	if (priority === 3) return 'high'
	if (priority === 2) return 'medium'
	return 'low'
}

function getSubtaskCount(task: TaskResponse): number {
	const related = task.related_tasks?.subtask
	return related ? related.length : 0
}

function getSubtaskDoneCount(task: TaskResponse): number {
	const related = task.related_tasks?.subtask
	return related ? related.filter(s => s.done).length : 0
}

function getInitials(name?: string): string {
	if (!name) return 'U'
	const parts = name.trim().split(' ')
	if (parts.length >= 2) return `${parts[0][0]}${parts[1][0]}`.toUpperCase()
	return name.substring(0, 2).toUpperCase()
}

function formatDueDate(dateStr?: string | null): string {
	if (!dateStr || dateStr.startsWith('0001-01-01')) return '-'
	try {
		const d = new Date(dateStr)
		return d.toLocaleDateString('pt-BR', {month: 'short', day: 'numeric'})
	} catch {
		return '-'
	}
}

async function createTask() {
	const title = newTaskTitle.value.trim()
	if (!title) return

	try {
		await createTaskMutation.mutateAsync({
			title,
			project_id: inboxProjectId.value,
		})
		newTaskTitle.value = ''
		showToast('Tarefa criada com sucesso!')
	} catch (e) {
		showToast('Erro ao criar tarefa')
	}
}

async function toggleTaskDone(task: TaskResponse) {
	try {
		await updateTaskMutation.mutateAsync({
			...task,
			done: !task.done,
		})
		showToast(!task.done ? 'Tarefa concluída' : 'Tarefa reaberta')
	} catch (e) {
		showToast('Erro ao atualizar tarefa')
	}
}

async function deleteTask(task: TaskResponse) {
	try {
		await deleteTaskMutation.mutateAsync(task.id)
		showToast('Tarefa movida para a lixeira')
	} catch (e) {
		showToast('Erro ao excluir tarefa')
	}
}

function openTaskDetails(task: TaskResponse) {
	router.push({
		name: 'task.detail',
		params: {id: task.id},
		state: {backdropView: router.currentRoute.value.fullPath},
	})
}

function focusInput() {
	taskInputRef.value?.focus()
}

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
}

function setFilter(filter: 'all' | 'pending' | 'completed') {
	activeFilter.value = filter
	showFilterMenu.value = false
}

function toggleRowMenu(taskId: number) {
	activeRowMenuId.value = activeRowMenuId.value === taskId ? null : taskId
}

function handleGlobalClick(e: MouseEvent) {
	const target = e.target as HTMLElement
	if (!target.closest('.dropdown-wrapper')) {
		showSortMenu.value = false
		showFilterMenu.value = false
	}
	if (!target.closest('.row-actions')) {
		activeRowMenuId.value = null
	}
}

onMounted(() => {
	document.addEventListener('click', handleGlobalClick)
})

onUnmounted(() => {
	document.removeEventListener('click', handleGlobalClick)
})
</script>

<style scoped lang="scss">
.inbox-page {
	width: 100%;
}

.toolbar {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 12px 24px;
	border-bottom: 1px solid var(--border-color);
	background: var(--bg-app);
}

.tabs-container {
	display: flex;
	gap: 8px;
}

.tab-btn {
	background: transparent;
	border: none;
	cursor: pointer;
	padding: 6px 12px;
	border-radius: 6px;
	font-size: 13px;
	font-weight: 500;
	color: var(--text-secondary);
	transition: background 0.15s, color 0.15s;

	&:hover {
		background: var(--bg-hover);
		color: var(--text-primary);
	}

	&.active {
		background: var(--bg-hover);
		color: var(--text-primary);
		font-weight: 600;
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
	padding: 6px 10px;
	font-size: 12px;
	font-weight: 500;
	color: var(--text-secondary);
	cursor: pointer;
	box-shadow: var(--shadow-card);
	transition: background 0.15s, color 0.15s;

	&:hover {
		background: var(--bg-hover);
		color: var(--text-primary);
	}
}

.dropdown-menu {
	position: absolute;
	right: 0;
	top: calc(100% + 4px);
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 6px;
	box-shadow: var(--shadow-dropdown);
	z-index: 100;
	min-width: 140px;
	overflow: hidden;
}

.dropdown-item {
	padding: 8px 12px;
	font-size: 13px;
	color: var(--text-primary);
	cursor: pointer;
	transition: background 0.1s;

	&:hover {
		background: var(--bg-hover);
	}

	&.danger {
		color: var(--priority-urgent);
	}
}

.main-card-wrapper {
	padding: 24px;
	display: flex;
	justify-content: center;
}

.tasks-card {
	width: 100%;
	max-width: 1000px;
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 8px;
	box-shadow: var(--shadow-card);
	overflow: hidden;
}

.task-input-container {
	display: flex;
	align-items: center;
	padding: 8px 16px;
	border-bottom: 1px solid var(--border-color);
	gap: 12px;
	transition: border-color 0.15s;

	&:focus-within {
		border-bottom-color: var(--accent-base);

		.add-btn {
			opacity: 1;
			pointer-events: auto;
		}
	}
}

.input-icon {
	color: var(--text-tertiary);
	display: flex;
	align-items: center;
}

.task-input {
	flex: 1;
	border: none;
	outline: none;
	background: transparent;
	font-size: 14px;
	color: var(--text-primary);
	font-family: inherit;

	&::placeholder {
		color: var(--text-tertiary);
	}
}

.add-btn {
	background: var(--accent-base);
	color: #ffffff;
	border: none;
	border-radius: 6px;
	padding: 4px 12px;
	font-size: 12px;
	font-weight: 500;
	cursor: pointer;
	opacity: 0;
	pointer-events: none;
	transition: opacity 0.15s, background 0.15s;

	&:hover {
		background: var(--accent-hover);
	}
}

.table-container {
	width: 100%;
}

.grid-row {
	display: grid;
	grid-template-columns: 32px minmax(300px, 1fr) 140px 100px 100px 80px 40px;
	gap: 16px;
	align-items: center;
	padding: 8px 16px;
	border-bottom: 1px solid var(--border-subtle);

	&:last-child {
		border-bottom: none;
	}
}

.table-header-row {
	background: var(--bg-header-row);
	border-bottom: 1px solid var(--border-color);
	font-size: 11px;
	font-weight: 600;
	color: var(--text-secondary);
	letter-spacing: 0.5px;
	user-select: none;
}

.task-row {
	cursor: pointer;
	transition: background 0.1s;

	&:hover {
		background: var(--bg-hover);

		.row-actions .action-btn {
			opacity: 1;
		}
	}

	&.completed {
		opacity: 0.5;

		.task-title-text {
			text-decoration: line-through;
		}
	}
}

.col-check {
	display: flex;
	align-items: center;
}

.task-checkbox {
	appearance: none;
	width: 16px;
	height: 16px;
	border: 1.5px solid var(--text-tertiary);
	border-radius: 4px;
	cursor: pointer;
	position: relative;
	background: transparent;
	transition: all 0.15s;

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
			border: solid #ffffff;
			border-width: 0 2px 2px 0;
			transform: rotate(45deg);
		}
	}
}

.col-title {
	display: flex;
	align-items: center;
	gap: 8px;
	min-width: 0;
	font-size: 13px;
	font-weight: 500;
	color: var(--text-primary);
}

.task-title-text {
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.priority-icon {
	display: inline-flex;
	align-items: center;
	justify-content: center;
	width: 14px;
	height: 14px;
	flex-shrink: 0;

	svg {
		width: 14px;
		height: 14px;
	}

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

.labels-group {
	display: flex;
	gap: 4px;
	margin-left: 4px;
}

.label-badge {
	font-size: 10px;
	font-weight: 500;
	padding: 1px 6px;
	border-radius: 4px;
	white-space: nowrap;
}

.col-project .project-pill {
	display: inline-flex;
	align-items: center;
	gap: 6px;
	padding: 2px 8px;
	background: #f0f0f0;
	border-radius: 12px;
	font-size: 11px;
	font-weight: 500;
	color: var(--text-secondary);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;

	:global(html.dark) & {
		background: #262a36;
	}
}

.project-dot {
	width: 6px;
	height: 6px;
	border-radius: 50%;
	flex-shrink: 0;
}

.col-date {
	font-size: 12px;
	color: var(--text-secondary);
	white-space: nowrap;
}

.col-assignee {
	display: flex;
	align-items: center;
}

.avatar-mini {
	width: 20px;
	height: 20px;
	border-radius: 50%;
	background: #e0e7ff;
	color: var(--accent-base);
	font-size: 9px;
	font-weight: 600;
	display: flex;
	align-items: center;
	justify-content: center;
}

.text-muted-dash {
	color: var(--text-tertiary);
	font-size: 13px;
}

.col-subtasks .subtask-indicator {
	display: flex;
	align-items: center;
	gap: 4px;
	font-size: 11px;
	color: var(--text-secondary);
}

.col-actions {
	position: relative;
	display: flex;
	justify-content: flex-end;
}

.row-actions {
	position: relative;

	.action-btn {
		background: transparent;
		border: none;
		cursor: pointer;
		color: var(--text-tertiary);
		font-size: 14px;
		padding: 2px 6px;
		border-radius: 4px;
		opacity: 0;
		transition: opacity 0.15s, background 0.15s, color 0.15s;

		&:hover {
			background: var(--bg-hover);
			color: var(--text-primary);
		}
	}
}

.row-menu {
	top: 100%;
	right: 0;
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
	border-radius: 50%;
	background: var(--bg-hover);
	color: var(--text-tertiary);
	display: flex;
	align-items: center;
	justify-content: center;
	margin-bottom: 16px;
}

.empty-title {
	font-size: 16px;
	font-weight: 600;
	color: var(--text-primary);
	margin-bottom: 6px;
}

.empty-text {
	font-size: 13px;
	color: var(--text-secondary);
}

.empty-link {
	color: var(--accent-base);
	font-weight: 500;
	cursor: pointer;
	margin-left: 4px;

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
