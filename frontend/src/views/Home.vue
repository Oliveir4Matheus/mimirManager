<template>
	<div class="overview-page">

		<!-- Content Area -->
		<main class="content-area">
			<div class="overview-wrapper">
				<!-- Greeting -->
				<h1 class="greeting">
					{{ greetingText }}
				</h1>

				<!-- Quick Add Task Input -->
				<div class="task-input-container">
					<div class="task-input-icon">
						<svg
							width="18"
							height="18"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
						>
							<line
								x1="8"
								y1="6"
								x2="21"
								y2="6"
							/>
							<line
								x1="8"
								y1="12"
								x2="21"
								y2="12"
							/>
							<line
								x1="8"
								y1="18"
								x2="21"
								y2="18"
							/>
							<line
								x1="3"
								y1="6"
								x2="3.01"
								y2="6"
							/>
							<line
								x1="3"
								y1="12"
								x2="3.01"
								y2="12"
							/>
							<line
								x1="3"
								y1="18"
								x2="3.01"
								y2="18"
							/>
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
						@click="createTask"
					>
						<svg
							width="14"
							height="14"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
						>
							<line
								x1="12"
								y1="5"
								x2="12"
								y2="19"
							/>
							<line
								x1="5"
								y1="12"
								x2="19"
								y2="12"
							/>
						</svg>
						ADD
					</button>
				</div>

				<!-- Last Viewed Section -->
				<section class="overview-section">
					<h2 class="section-title">
						Last viewed
					</h2>
					<div class="last-viewed-grid">
						<div
							v-for="card in lastViewedCards"
							:key="card.id"
							class="last-viewed-card"
							tabindex="0"
							:aria-label="'Abrir ' + card.title"
							@click="openLastViewed(card)"
							@keydown.enter="openLastViewed(card)"
						>
							<div class="last-viewed-title">
								{{ card.title }}
							</div>
						</div>
					</div>
				</section>

				<!-- Current Tasks Section -->
				<section class="overview-section">
					<h2 class="section-title">
						Current Tasks
					</h2>

					<!-- Tasks List (Populated) -->
					<div
						v-if="tasks.length > 0"
						class="tasks-list"
					>
						<div
							v-for="task in tasks"
							:key="task.id"
							class="task-row"
							:class="{completed: task.completed}"
							@click="onTaskRowClick(task, $event)"
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

							<!-- Task Title & Priority -->
							<div class="cell-title">
								<span
									class="priority-icon"
									:class="'priority-' + task.priority"
									:title="'Prioridade: ' + task.priority"
								>
									<!-- Urgent: Triangle -->
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
									<!-- High: Tall double bars -->
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
									<!-- Medium: Plus / Bar -->
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
									<!-- Low: Small bars -->
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
						</div>
					</div>

					<!-- Empty State View (Llama Mascot) -->
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
							Crie uma tarefa acima para começar.
						</p>
					</div>
				</section>
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

		<!-- Dev Controls (Conforme item 5.1/7 dos Critérios de Aceitação da Spec) -->
		<div class="dev-controls">
			<button
				type="button"
				class="dev-btn"
				@click="clearAllTasks"
			>
				Simular Estado Vazio
			</button>
			<button
				type="button"
				class="dev-btn"
				@click="restoreData"
			>
				Restaurar Dados
			</button>
		</div>
	</div>
</template>

<script setup lang="ts">
import {computed, ref} from 'vue'
import {useRouter} from 'vue-router'
import {useAuthStore} from '@/stores/auth'
import {useDaytimeSalutation} from '@/composables/useDaytimeSalutation'
import Icon from '@/components/misc/Icon'

export interface OverviewTask {
	id: number
	title: string
	project: string
	projectColor: string
	priority: 'urgent' | 'high' | 'medium' | 'low'
	completed: boolean
}

const router = useRouter()
const authStore = useAuthStore()
const salutationComposable = useDaytimeSalutation()

const taskInputRef = ref<HTMLInputElement | null>(null)
const newTaskTitle = ref('')

const userInitials = computed(() => {
	const name = authStore.userDisplayName || 'Matheus'
	return name.substring(0, 2).toUpperCase()
})

const greetingText = computed(() => {
	const user = authStore.userDisplayName || 'Matheus'
	if (salutationComposable.value) {
		return `${salutationComposable.value}, ${user}`
	}
	return `One more thing, ${user}?`
})

const lastViewedCards = ref([
	{id: 'inbox', title: 'Inbox', route: '/inbox'},
])

const initialTasks: OverviewTask[] = [
	{
		id: 1,
		title: 'Validar requisitos',
		project: 'Inbox',
		projectColor: '#9ca3af',
		priority: 'medium',
		completed: false,
	},
	{
		id: 2,
		title: 'Preparar entrega da equipe',
		project: 'Inbox',
		projectColor: '#9ca3af',
		priority: 'high',
		completed: false,
	},
	{
		id: 3,
		title: 'Revisar proposta de design',
		project: 'Inbox',
		projectColor: '#9ca3af',
		priority: 'urgent',
		completed: false,
	},
	{
		id: 4,
		title: 'Planejar novo projeto',
		project: 'Inbox',
		projectColor: '#9ca3af',
		priority: 'low',
		completed: false,
	},
]

const tasks = ref<OverviewTask[]>(JSON.parse(JSON.stringify(initialTasks)))

const toastMessage = ref('')
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(msg: string) {
	toastMessage.value = msg
	if (toastTimer) clearTimeout(toastTimer)
	toastTimer = setTimeout(() => {
		toastMessage.value = ''
	}, 3000)
}

function createTask() {
	const title = newTaskTitle.value.trim()
	if (!title) return

	tasks.value.unshift({
		id: Date.now(),
		title,
		project: 'Inbox',
		projectColor: '#9ca3af',
		priority: 'medium',
		completed: false,
	})
	newTaskTitle.value = ''
	showToast('Tarefa criada com sucesso')
}

function toggleComplete(id: number) {
	const task = tasks.value.find(t => t.id === id)
	if (task) {
		task.completed = !task.completed
		showToast(task.completed ? 'Tarefa concluída' : 'Tarefa reaberta')
	}
}

function onTaskRowClick(task: OverviewTask, event: MouseEvent) {
	const target = event.target as HTMLElement
	if (target.closest('.task-checkbox')) return
	showToast(`Abrindo detalhes: ${task.title}`)
}

function openLastViewed(card: {id: string; title: string; route: string}) {
	showToast(`Navegando para ${card.title}...`)
	router.push(card.route)
}

function clearAllTasks() {
	tasks.value = []
	showToast('Lista esvaziada')
}

function restoreData() {
	tasks.value = JSON.parse(JSON.stringify(initialTasks))
	showToast('Dados restaurados')
}
</script>

<style scoped lang="scss">
.overview-page {
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

:global(html.dark) .overview-page {
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
	padding: 48px 16px;
	overflow-y: auto;
	display: flex;
	justify-content: center;
}

.overview-wrapper {
	width: 100%;
	max-width: 800px;
	display: flex;
	flex-direction: column;
}

.greeting {
	font-size: 24px;
	font-weight: 600;
	letter-spacing: -0.5px;
	margin-bottom: 24px;
	color: var(--text-primary);
}

/* Quick Add Task Input */
.task-input-container {
	display: flex;
	align-items: center;
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 8px;
	padding: 8px 12px 8px 16px;
	box-shadow: var(--shadow-card);
	margin-bottom: 48px;
	transition: border-color 0.15s, box-shadow 0.15s;

	&:focus-within {
		border-color: var(--accent-base);
		box-shadow: 0 0 0 3px rgba(94, 106, 210, 0.1);
	}
}

.task-input-icon {
	color: var(--text-tertiary);
	display: flex;
	align-items: center;
	margin-right: 12px;
}

.task-input {
	flex: 1;
	border: none;
	background: transparent;
	font-size: 14px;
	color: var(--text-primary);
	outline: none;
	padding: 4px 0;
	font-family: var(--font-family);

	&::placeholder {
		color: var(--text-tertiary);
	}
}

.add-btn {
	background: var(--accent-base);
	color: #ffffff;
	border: none;
	border-radius: 6px;
	padding: 6px 16px;
	font-size: 13px;
	font-weight: 500;
	cursor: pointer;
	display: flex;
	align-items: center;
	gap: 4px;
	transition: background 0.15s, opacity 0.15s;
	opacity: 0.5;

	.task-input-container:focus-within & {
		opacity: 1;
	}

	&:hover {
		background: var(--accent-hover);
	}
}

/* Sections */
.overview-section {
	margin-bottom: 48px;

	&:last-child {
		margin-bottom: 0;
	}
}

.section-title {
	font-size: 16px;
	font-weight: 600;
	margin-bottom: 16px;
	color: var(--text-primary);
}

/* Last Viewed */
.last-viewed-grid {
	display: flex;
	gap: 16px;
	flex-wrap: wrap;
}

.last-viewed-card {
	width: 160px;
	height: 100px;
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 8px;
	padding: 16px;
	display: flex;
	flex-direction: column;
	justify-content: flex-end;
	cursor: pointer;
	box-shadow: var(--shadow-card);
	transition: border-color 0.15s, transform 0.15s;
	outline: none;

	&:hover,
	&:focus-visible {
		border-color: var(--accent-base);
		transform: translateY(-2px);
	}
}

.last-viewed-title {
	font-size: 14px;
	font-weight: 500;
	color: var(--text-primary);
}

/* Current Tasks List */
.tasks-list {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 8px;
	box-shadow: var(--shadow-card);
	display: flex;
	flex-direction: column;
	overflow: hidden;
}

.task-row {
	display: grid;
	grid-template-columns: 32px 100px 1fr;
	align-items: center;
	padding: 10px 16px;
	border-bottom: 1px solid var(--border-subtle);
	transition: background 0.1s;
	cursor: pointer;

	&:last-child {
		border-bottom: none;
	}

	&:hover {
		background: var(--bg-hover);
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

/* Empty State */
.empty-state-view {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 64px 24px;
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
