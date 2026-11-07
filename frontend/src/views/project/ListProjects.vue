<template>
	<div class="projects-page">
		<!-- Top Bar / Breadcrumb & Header Actions -->
		<header class="top-header">
			<div class="breadcrumb">
				<span class="breadcrumb-item">MimirManager</span>
				<span class="breadcrumb-separator">/</span>
				<span class="breadcrumb-current">Projects</span>
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
			<div class="projects-wrapper">
				<!-- Toolbar (Filtros e Ações) -->
				<div class="projects-toolbar">
					<label class="checkbox-label">
						<input
							v-model="showArchived"
							type="checkbox"
							@change="showToast('Filtro alterado: Mostrar Arquivados')"
						>
						<span>Show Archived</span>
					</label>

					<div class="action-buttons">
						<button
							type="button"
							class="btn btn-secondary"
							@click="createNew('filter')"
						>
							<svg
								width="14"
								height="14"
								viewBox="0 0 24 24"
								fill="none"
								stroke="currentColor"
								stroke-width="2"
							>
								<polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3" />
							</svg>
							New Saved Filter
						</button>
						<button
							type="button"
							class="btn btn-primary"
							@click="createNew('project')"
						>
							<svg
								width="14"
								height="14"
								viewBox="0 0 24 24"
								fill="none"
								stroke="currentColor"
								stroke-width="2"
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
							New Project
						</button>
					</div>
				</div>

				<!-- Grid de Cards Preenchido -->
				<div
					v-if="displayedItems.length > 0"
					class="grid-container"
				>
					<div
						v-for="item in displayedItems"
						:key="item.id"
						class="project-card"
						tabindex="0"
						role="button"
						:aria-label="'Abrir ' + item.title"
						@click="onCardClick(item)"
						@keydown.enter="onCardClick(item)"
					>
						<div class="card-title">
							<span
								v-if="item.type === 'filter'"
								class="card-icon"
							>
								<svg
									width="16"
									height="16"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2"
								>
									<polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3" />
								</svg>
							</span>
							<span>{{ item.title }}</span>
						</div>
					</div>
				</div>

				<!-- Estado Vazio -->
				<div
					v-else
					class="empty-state"
				>
					<div class="empty-icon">
						<svg
							width="24"
							height="24"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
						>
							<path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z" />
						</svg>
					</div>
					<h3 class="empty-title">
						Nenhum projeto encontrado
					</h3>
					<p class="empty-text">
						Crie seu primeiro projeto para começar a organizar suas tarefas.
					</p>
					<button
						type="button"
						class="btn btn-primary"
						@click="createNew('project')"
					>
						Criar Projeto
					</button>
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
			<button
				type="button"
				class="dev-btn"
				@click="resetData"
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
import Icon from '@/components/misc/Icon'

export interface ProjectItem {
	id: number
	title: string
	type: 'project' | 'filter'
	icon?: string | null
	isArchived?: boolean
}

const router = useRouter()
const authStore = useAuthStore()

const userInitials = computed(() => {
	const name = authStore.userDisplayName || 'Matheus'
	return name.substring(0, 2).toUpperCase()
})

const showArchived = ref(false)
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

const initialData: ProjectItem[] = [
	{id: 1, title: 'Inbox', type: 'project', icon: null, isArchived: false},
	{id: 2, title: 'My Open Tasks', type: 'filter', icon: 'filter', isArchived: false},
	{id: 3, title: 'Marketing Q4', type: 'project', icon: null, isArchived: false},
	{id: 4, title: 'Backend API', type: 'project', icon: null, isArchived: false},
]

const items = ref<ProjectItem[]>(JSON.parse(JSON.stringify(initialData)))

const displayedItems = computed(() => {
	if (isSimulatedEmpty.value) return []
	return items.value.filter(item => showArchived.value || !item.isArchived)
})

function onCardClick(item: ProjectItem) {
	showToast(`Navegando para: ${item.title}`)
	if (item.type === 'project' && item.id === 1) {
		router.push('/inbox')
	}
}

function createNew(type: 'project' | 'filter') {
	const name = window.prompt(`Nome do novo ${type === 'project' ? 'Projeto' : 'Filtro Salvo'}:`)
	if (!name) return

	items.value.unshift({
		id: Date.now(),
		title: name,
		type,
		icon: type === 'filter' ? 'filter' : null,
		isArchived: false,
	})
	showToast(`${type === 'project' ? 'Projeto' : 'Filtro'} criado com sucesso!`)
}

function toggleEmptyMode() {
	isSimulatedEmpty.value = !isSimulatedEmpty.value
	showToast(isSimulatedEmpty.value ? 'Visualizando Estado Vazio' : 'Visualizando Grid Preenchido')
}

function resetData() {
	items.value = JSON.parse(JSON.stringify(initialData))
	isSimulatedEmpty.value = false
	showToast('Dados restaurados')
}
</script>

<style scoped lang="scss">
.projects-page {
	/* Design Tokens - Light Theme */
	--font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
	--bg-app: #f9f9f9;
	--bg-sidebar: #fcfcfc;
	--bg-card: #ffffff;
	--bg-hover: #f4f4f5;

	--text-primary: #111111;
	--text-secondary: #6b6b6b;
	--text-tertiary: #9ca3af;

	--border-color: #e5e5e5;
	--border-focus: #5e6ad2;

	--accent-base: #5e6ad2;
	--accent-hover: #4a55b5;

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

:global(html.dark) .projects-page {
	/* Design Tokens - Dark Theme */
	--bg-app: #0f0f0f;
	--bg-sidebar: #0f0f0f;
	--bg-card: #1c1c1c;
	--bg-hover: #2a2a2a;

	--text-primary: #e5e5e5;
	--text-secondary: #8a8a8a;
	--text-tertiary: #555555;

	--border-color: #333333;
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
	padding: 32px 24px;
	overflow-y: auto;
	display: flex;
	flex-direction: column;
	align-items: center;
}

.projects-wrapper {
	width: 100%;
	max-width: 1200px;
	display: flex;
	flex-direction: column;
}

/* Toolbar */
.projects-toolbar {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 24px;
	flex-wrap: wrap;
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
}

.action-buttons {
	display: flex;
	gap: 12px;
}

.btn {
	display: flex;
	align-items: center;
	gap: 6px;
	border-radius: 6px;
	padding: 6px 12px;
	font-size: 12px;
	font-weight: 600;
	cursor: pointer;
	text-transform: uppercase;
	letter-spacing: 0.5px;
	transition: all 0.15s ease;
}

.btn-secondary {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	color: var(--text-secondary);

	&:hover {
		background: var(--bg-hover);
		border-color: var(--text-tertiary);
		color: var(--text-primary);
	}
}

.btn-primary {
	background: var(--accent-base);
	border: 1px solid var(--accent-base);
	color: #ffffff;

	&:hover {
		background: var(--accent-hover);
		border-color: var(--accent-hover);
	}
}

/* Grid */
.grid-container {
	display: grid;
	grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
	gap: 16px;
}

.project-card {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 8px;
	padding: 24px;
	height: 140px;
	display: flex;
	flex-direction: column;
	justify-content: flex-end;
	cursor: pointer;
	box-shadow: var(--shadow-card);
	transition: border-color 0.15s, transform 0.15s, box-shadow 0.15s;
	outline: none;

	&:hover,
	&:focus-visible {
		border-color: var(--accent-base);
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(94, 106, 210, 0.08);
	}
}

.card-title {
	font-size: 16px;
	font-weight: 500;
	color: var(--text-primary);
	display: flex;
	align-items: center;
	gap: 8px;
}

.card-icon {
	color: var(--text-secondary);
	display: flex;
	align-items: center;
}

/* Empty State */
.empty-state {
	display: flex;
	flex-direction: column;
	align-items: center;
	padding: 64px 24px;
	border: 1px dashed var(--border-color);
	border-radius: 8px;
	margin-top: 16px;
	background: var(--bg-card);
	text-align: center;
}

.empty-icon {
	width: 48px;
	height: 48px;
	border-radius: 50%;
	background: var(--bg-hover);
	color: var(--text-secondary);
	display: flex;
	align-items: center;
	justify-content: center;
	margin-bottom: 16px;
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
	margin-bottom: 24px;
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
