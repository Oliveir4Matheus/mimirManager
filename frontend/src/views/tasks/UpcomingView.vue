<template>
	<div class="upcoming-page">
		<!-- Top Bar / Breadcrumb & Header Actions -->
		<header class="top-header">
			<div class="breadcrumb">
				<span class="breadcrumb-item">MimirManager</span>
				<span class="breadcrumb-separator">/</span>
				<span class="breadcrumb-current">Upcoming</span>
			</div>
			<div class="header-actions">
				<button
					type="button"
					class="icon-btn"
					title="Pesquisar"
					@click="showToast('Pesquisa rápida')"
				>
					<Icon icon="search" />
				</button>
				<button
					type="button"
					class="icon-btn"
					title="Notificações"
					@click="showToast('Nenhuma notificação nova')"
				>
					<Icon icon="bell" />
				</button>
				<div
					class="user-profile"
					@click="showToast('Perfil do usuário')"
				>
					<div class="avatar">
						{{ userInitials }}
					</div>
					<span class="username">{{ authStore.userDisplayName || 'Matheus' }}</span>
				</div>
			</div>
		</header>

		<!-- Content Area -->
		<main class="content-area">
			<div class="upcoming-wrapper">
				<!-- Header Section -->
				<div class="view-header">
					<h1 class="view-title">
						{{ titleText }}
					</h1>
					<div class="view-controls">
						<button
							type="button"
							class="btn-secondary"
							@click="selectDateRange"
						>
							Select a date range
						</button>
						<div class="checkbox-group">
							<label class="checkbox-label">
								<input
									v-model="showTasksWithoutDate"
									type="checkbox"
									@change="showToast('Filtro: Tarefas sem data atualizado')"
								>
								<span>Show tasks without date</span>
							</label>
							<label class="checkbox-label">
								<input
									v-model="showOverdueTasks"
									type="checkbox"
									@change="showToast('Filtro: Tarefas atrasadas atualizado')"
								>
								<span>Show overdue tasks</span>
							</label>
						</div>
					</div>
				</div>

				<!-- Table Container -->
				<div
					v-if="filteredTasks.length > 0"
					class="table-container"
				>
					<!-- Header Row -->
					<div class="table-header-row">
						<div><!-- Checkbox column --></div>
						<div>Tarefa</div>
						<div>Projeto</div>
						<div>Data de entrega</div>
						<div>Responsável</div>
						<div>Subtarefas</div>
						<div><!-- Actions column --></div>
					</div>

					<!-- Task Rows -->
					<div
						v-for="task in filteredTasks"
						:key="task.id"
						class="grid-row"
						:class="{completed: task.completed}"
						@click="onRowClick(task, $event)"
					>
						<!-- Checkbox -->
						<div class="cell-checkbox">
							<input
								type="checkbox"
								class="task-checkbox"
								:checked="task.completed"
								:aria-label="'Concluir tarefa: ' + task.title"
								@click.stop
								@change="toggleComplete(task.id)"
							>
						</div>

						<!-- Task Title & Priority -->
						<div class="cell-title">
							<span
								class="priority-icon"
								:class="'priority-' + task.priority"
								:title="'Prioridade: ' + task.priority"
							>
								<!-- Urgent -->
								<svg
									v-if="task.priority === 'urgent'"
									viewBox="0 0 24 24"
									fill="currentColor"
								>
									<path
										d="M12 2L2 22h20L12 2z"
										fill="none"
										stroke="currentColor"
										stroke-width="2"
									/>
									<line
										x1="12"
										y1="9"
										x2="12"
										y2="15"
										stroke="currentColor"
										stroke-width="2"
										stroke-linecap="round"
									/>
									<circle
										cx="12"
										cy="18"
										r="1"
										fill="currentColor"
									/>
								</svg>
								<!-- High -->
								<svg
									v-else-if="task.priority === 'high'"
									viewBox="0 0 24 24"
									fill="currentColor"
								>
									<rect
										x="6"
										y="10"
										width="4"
										height="10"
										rx="1"
									/>
									<rect
										x="14"
										y="4"
										width="4"
										height="16"
										rx="1"
									/>
								</svg>
								<!-- Medium -->
								<svg
									v-else-if="task.priority === 'medium'"
									viewBox="0 0 24 24"
									fill="currentColor"
								>
									<path
										d="M12 4v16m-8-8h16"
										stroke="currentColor"
										stroke-width="2"
										stroke-linecap="round"
									/>
								</svg>
								<!-- Low -->
								<svg
									v-else
									viewBox="0 0 24 24"
									fill="currentColor"
								>
									<rect
										x="6"
										y="14"
										width="4"
										height="6"
										rx="1"
									/>
									<rect
										x="14"
										y="14"
										width="4"
										height="6"
										rx="1"
									/>
								</svg>
							</span>
							<span class="task-title-text">{{ task.title }}</span>
						</div>

						<!-- Project Pill -->
						<div class="cell-project">
							<span class="project-pill">
								<span
									class="project-dot"
									:style="{backgroundColor: task.projectColor}"
								/>
								{{ task.project }}
							</span>
						</div>

						<!-- Due Date -->
						<div class="cell-date">
							{{ task.dueDate || '-' }}
						</div>

						<!-- Assignee -->
						<div class="cell-assignee">
							<div
								v-if="task.assignee"
								class="table-avatar"
								:title="task.assignee"
							>
								{{ task.assignee.substring(0, 2).toUpperCase() }}
							</div>
							<span
								v-else
								class="text-muted"
							>-</span>
						</div>

						<!-- Subtasks -->
						<div class="cell-subtasks">
							<span
								v-if="task.subtasks"
								class="subtask-badge"
							>
								<svg
									width="12"
									height="12"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2"
								>
									<polyline points="9 11 12 14 22 4" />
									<path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11" />
								</svg>
								{{ task.subtasks }}
							</span>
							<span
								v-else
								class="text-muted"
							>-</span>
						</div>

						<!-- Actions Menu -->
						<div class="cell-actions">
							<button
								type="button"
								class="row-action-btn"
								title="Ações"
								@click.stop="showToast('Menu de opções para: ' + task.title)"
							>
								•••
							</button>
						</div>
					</div>
				</div>

				<!-- Empty State View -->
				<div
					v-else
					class="empty-state-view"
				>
					<div class="empty-icon-circle">
						<div class="llama-mockup">
							🦙
						</div>
					</div>
					<h3 class="empty-title">
						Nothing to do — Have a nice day!
					</h3>
					<p class="empty-subtitle">
						Nenhuma tarefa encontrada para este período.
					</p>
				</div>
			</div>
		</main>

		<!-- Floating Toast Notification -->
		<div
			v-if="toastMessage"
			class="toast-notification"
		>
			<svg
				width="16"
				height="16"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
			>
				<polyline points="20 6 9 17 4 12" />
			</svg>
			<span>{{ toastMessage }}</span>
		</div>

		<!-- Dev Controls -->
		<div class="dev-controls">
			<button
				type="button"
				class="dev-btn"
				@click="toggleEmptyMode"
			>
				Alternar Vazio / Preenchido
			</button>
		</div>
	</div>
</template>

<script setup lang="ts">
import {computed, ref} from 'vue'
import {useAuthStore} from '@/stores/auth'
import Icon from '@/components/misc/Icon'

export interface UpcomingTask {
	id: number
	title: string
	project: string
	projectColor: string
	priority: 'urgent' | 'high' | 'medium' | 'low'
	dueDate: string
	assignee: string
	subtasks: string
	completed: boolean
}

const authStore = useAuthStore()

const userInitials = computed(() => {
	const name = authStore.userDisplayName || 'Matheus'
	return name.substring(0, 2).toUpperCase()
})

const titleText = ref('Tasks from November 6, 2026 until November 13, 2026')
const showTasksWithoutDate = ref(false)
const showOverdueTasks = ref(false)
const isSimulatedEmpty = ref(false)

const toastMessage = ref('')
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(msg: string) {
	toastMessage.value = msg
	if (toastTimer) clearTimeout(toastTimer)
	toastTimer = setTimeout(() => {
		toastMessage.value = ''
	}, 3000)
}

const initialTasks: UpcomingTask[] = [
	{
		id: 1,
		title: 'Validar requisitos',
		project: 'Inbox',
		projectColor: '#9ca3af',
		priority: 'medium',
		dueDate: 'Nov 8, 2026',
		assignee: 'Matheus Oliveira',
		subtasks: '1/3',
		completed: false,
	},
	{
		id: 2,
		title: 'Preparar entrega da equipe',
		project: 'Inbox',
		projectColor: '#9ca3af',
		priority: 'high',
		dueDate: 'Nov 10, 2026',
		assignee: 'Matheus Oliveira',
		subtasks: '2/4',
		completed: false,
	},
	{
		id: 3,
		title: 'Revisar proposta de design',
		project: 'Inbox',
		projectColor: '#9ca3af',
		priority: 'urgent',
		dueDate: 'Nov 12, 2026',
		assignee: 'Matheus Oliveira',
		subtasks: '0/2',
		completed: false,
	},
	{
		id: 4,
		title: 'Planejar novo projeto',
		project: 'Inbox',
		projectColor: '#9ca3af',
		priority: 'low',
		dueDate: 'Nov 13, 2026',
		assignee: 'Matheus Oliveira',
		subtasks: '3/3',
		completed: false,
	},
]

const tasks = ref<UpcomingTask[]>(JSON.parse(JSON.stringify(initialTasks)))

const filteredTasks = computed(() => {
	if (isSimulatedEmpty.value) return []
	return tasks.value
})

function selectDateRange() {
	showToast('Abrindo seletor de intervalo de datas...')
}

function toggleComplete(id: number) {
	const task = tasks.value.find(t => t.id === id)
	if (task) {
		task.completed = !task.completed
		showToast(task.completed ? 'Tarefa concluída' : 'Tarefa reaberta')
	}
}

function onRowClick(task: UpcomingTask, event: MouseEvent) {
	const target = event.target as HTMLElement
	if (target.closest('.task-checkbox') || target.closest('.row-action-btn')) return
	showToast(`Abrindo detalhes: ${task.title}`)
}

function toggleEmptyMode() {
	isSimulatedEmpty.value = !isSimulatedEmpty.value
	showToast(isSimulatedEmpty.value ? 'Visualizando Estado Vazio' : 'Visualizando Lista Preenchida')
}
</script>

<style scoped lang="scss">
.upcoming-page {
	/* Design Tokens - Light Theme */
	--font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
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

	--shadow-card: 0 1px 3px rgba(0, 0, 0, 0.02), 0 1px 2px rgba(0, 0, 0, 0.04);
	--shadow-dropdown: 0 4px 12px rgba(0, 0, 0, 0.1);

	font-family: var(--font-family);
	background-color: var(--bg-app);
	color: var(--text-primary);
	min-height: 100vh;
	display: flex;
	flex-direction: column;
	width: 100%;
}

:global(html.dark) .upcoming-page {
	/* Design Tokens - Dark Theme */
	--bg-app: #0f0f0f;
	--bg-sidebar: #0f0f0f;
	--bg-card: #1c1c1c;
	--bg-hover: #262a36;
	--bg-header-row: #181b22;
	--bg-modal: #1c1c1c;

	--text-primary: #e5e5e5;
	--text-secondary: #8a8a8a;
	--text-tertiary: #555555;

	--border-color: #333333;
	--border-subtle: #222222;
}

/* Header */
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
		font-weight: 400;
	}

	.breadcrumb-current {
		color: var(--text-primary);
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
	align-items: center;
	justify-content: center;
	padding: 6px;
	border-radius: 6px;
	transition: background 0.15s, color 0.15s;

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
	padding: 2px 6px;
	border-radius: 6px;
	transition: background 0.15s;

	&:hover {
		background: var(--bg-hover);
	}

	.avatar {
		width: 24px;
		height: 24px;
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
}

/* Content Area */
.content-area {
	flex: 1;
	padding: 32px 16px;
	overflow-y: auto;
	display: flex;
	justify-content: center;
}

.upcoming-wrapper {
	width: 100%;
	max-width: 1000px;
	display: flex;
	flex-direction: column;
}

.view-header {
	display: flex;
	flex-direction: column;
	gap: 16px;
	margin-bottom: 24px;
}

.view-title {
	font-size: 24px;
	font-weight: 600;
	letter-spacing: -0.5px;
	color: var(--text-primary);
}

.view-controls {
	display: flex;
	align-items: center;
	gap: 24px;
	flex-wrap: wrap;
}

.btn-secondary {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 6px;
	padding: 6px 12px;
	font-size: 12px;
	font-weight: 600;
	color: var(--text-secondary);
	cursor: pointer;
	transition: background 0.15s, border-color 0.15s, color 0.15s;
	text-transform: uppercase;
	letter-spacing: 0.5px;

	&:hover {
		background: var(--bg-hover);
		border-color: var(--text-tertiary);
		color: var(--text-primary);
	}
}

.checkbox-group {
	display: flex;
	gap: 16px;
}

.checkbox-label {
	display: flex;
	align-items: center;
	gap: 8px;
	font-size: 13px;
	font-weight: 500;
	color: var(--text-secondary);
	cursor: pointer;

	input[type="checkbox"] {
		appearance: none;
		width: 14px;
		height: 14px;
		border: 1.5px solid var(--text-tertiary);
		border-radius: 4px;
		cursor: pointer;
		position: relative;
		transition: all 0.15s;
		background: transparent;

		&:hover {
			border-color: var(--accent-base);
		}

		&:checked {
			background: var(--accent-base);
			border-color: var(--accent-base);

			&::after {
				content: '';
				position: absolute;
				top: 1px;
				left: 3px;
				width: 3px;
				height: 6px;
				border: solid #ffffff;
				border-width: 0 2px 2px 0;
				transform: rotate(45deg);
			}
		}
	}
}

/* Table */
.table-container {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 8px;
	box-shadow: var(--shadow-card);
	display: flex;
	flex-direction: column;
	min-width: 800px;
}

.table-header-row,
.grid-row {
	display: grid;
	grid-template-columns: 32px minmax(250px, 1fr) 140px 100px 90px 70px 40px;
	align-items: center;
	padding: 8px 16px;
	gap: 16px;
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
	cursor: pointer;
	transition: background 0.1s;

	&:last-child {
		border-bottom: none;
		border-bottom-left-radius: 8px;
		border-bottom-right-radius: 8px;
	}

	&:hover {
		background: var(--bg-hover);

		.row-action-btn {
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

.cell-checkbox {
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
	transition: all 0.15s;
	background: transparent;

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

.cell-title {
	font-size: 13px;
	font-weight: 500;
	color: var(--text-primary);
	display: flex;
	align-items: center;
	gap: 8px;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.task-title-text {
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
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

.cell-project {
	display: flex;
	align-items: center;
}

.project-pill {
	font-size: 11px;
	font-weight: 500;
	padding: 2px 8px;
	border-radius: 12px;
	background: #f0f0f0;
	color: var(--text-secondary);
	display: inline-flex;
	align-items: center;
	gap: 6px;
	justify-self: start;

	:global(html.dark) & {
		background: #262a36;
	}
}

.project-dot {
	width: 6px;
	height: 6px;
	border-radius: 50%;
}

.cell-date {
	font-size: 12px;
	color: var(--text-secondary);
}

.cell-assignee {
	display: flex;
	align-items: center;
}

.table-avatar {
	width: 20px;
	height: 20px;
	border-radius: 50%;
	background: #e0e7ff;
	color: var(--accent-base);
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 10px;
	font-weight: 600;
}

.cell-subtasks {
	display: flex;
	align-items: center;
}

.subtask-badge {
	display: flex;
	align-items: center;
	gap: 4px;
	font-size: 12px;
	color: var(--text-secondary);
}

.text-muted {
	color: var(--text-tertiary);
	font-size: 12px;
}

.cell-actions {
	display: flex;
	align-items: center;
	justify-content: flex-end;
}

.row-action-btn {
	background: transparent;
	border: none;
	color: var(--text-secondary);
	font-size: 13px;
	cursor: pointer;
	padding: 2px 6px;
	border-radius: 4px;
	opacity: 0;
	transition: opacity 0.15s, background 0.15s;

	&:hover {
		background: var(--bg-hover);
		color: var(--text-primary);
	}
}

/* Empty State */
.empty-state-view {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 64px 24px;
	margin-top: 24px;
	text-align: center;
}

.empty-icon-circle {
	width: 120px;
	height: 120px;
	background: var(--accent-base);
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	margin-bottom: 24px;
	position: relative;
	overflow: hidden;
}

.llama-mockup {
	font-size: 60px;
	filter: grayscale(100%) contrast(200%) brightness(150%);
	transform: translateY(10px);
}

.empty-title {
	font-size: 16px;
	font-weight: 600;
	color: var(--text-primary);
	margin-bottom: 8px;
}

.empty-subtitle {
	font-size: 14px;
	color: var(--text-secondary);
}

/* Toast */
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
	display: flex;
	align-items: center;
	gap: 8px;
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

/* Dev Controls */
.dev-controls {
	position: fixed;
	bottom: 16px;
	right: 16px;
	background: var(--bg-card);
	padding: 10px 12px;
	border-radius: 8px;
	border: 1px solid var(--border-color);
	box-shadow: var(--shadow-dropdown);
	display: flex;
	gap: 8px;
	z-index: 100;
}

.dev-btn {
	font-size: 12px;
	padding: 4px 8px;
	cursor: pointer;
	background: var(--bg-hover);
	color: var(--text-secondary);
	border: 1px solid var(--border-color);
	border-radius: 4px;
	transition: all 0.15s;

	&:hover {
		background: var(--accent-base);
		color: #ffffff;
		border-color: var(--accent-base);
	}
}
</style>
