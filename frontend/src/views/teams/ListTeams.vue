<template>
	<div class="teams-page">
		<!-- Top Bar / Breadcrumb & Header Actions -->
		<header class="top-header">
			<div class="breadcrumb">
				<span class="breadcrumb-item">MimirManager</span>
				<span class="breadcrumb-separator">/</span>
				<span class="breadcrumb-current">Teams</span>
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
			<div class="teams-wrapper">
				<!-- Header Section -->
				<div class="view-header">
					<h1 class="view-title">
						Teams
					</h1>
					<button
						type="button"
						class="btn-primary"
						@click="openModal()"
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
						Create a Team
					</button>
				</div>

				<!-- Teams Table Container -->
				<div
					v-if="displayedTeams.length > 0"
					class="teams-table"
				>
					<div
						v-for="team in displayedTeams"
						:key="team.id"
						class="team-row"
						@click="onRowClick(team)"
					>
						<!-- Name with icon -->
						<div class="cell-name">
							<svg
								class="team-icon"
								width="16"
								height="16"
								viewBox="0 0 24 24"
								fill="none"
								stroke="currentColor"
								stroke-width="2"
							>
								<path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2" />
								<circle
									cx="9"
									cy="7"
									r="4"
								/>
								<path d="M23 21v-2a4 4 0 0 0-3-3.87" />
								<path d="M16 3.13a4 4 0 0 1 0 7.75" />
							</svg>
							<span class="name-text">{{ team.name }}</span>
						</div>

						<!-- Description -->
						<div class="cell-desc">
							{{ team.description || '-' }}
						</div>

						<!-- Members Count -->
						<div class="cell-members">
							<svg
								width="14"
								height="14"
								viewBox="0 0 24 24"
								fill="none"
								stroke="currentColor"
								stroke-width="2"
							>
								<path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2" />
								<circle
									cx="12"
									cy="7"
									r="4"
								/>
							</svg>
							<span>{{ team.memberCount }}</span>
						</div>

						<!-- Actions -->
						<div class="cell-actions">
							<button
								type="button"
								class="action-btn"
								title="Configurações"
								:aria-label="'Configurações da equipe ' + team.name"
								@click.stop="openModal(team.id)"
							>
								<svg
									width="14"
									height="14"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2"
								>
									<path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7" />
									<path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z" />
								</svg>
							</button>
							<button
								type="button"
								class="action-btn danger"
								title="Excluir"
								:aria-label="'Excluir equipe ' + team.name"
								@click.stop="deleteTeam(team.id)"
							>
								<svg
									width="14"
									height="14"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2"
								>
									<polyline points="3 6 5 6 21 6" />
									<path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2" />
								</svg>
							</button>
						</div>
					</div>
				</div>

				<!-- Empty State -->
				<div
					v-else
					class="empty-state-view"
				>
					<p class="empty-text">
						You are currently not part of any teams.
						<a
							class="empty-link"
							@click="openModal()"
						>Create a team.</a>
					</p>
				</div>
			</div>
		</main>

		<!-- Modal Criar / Editar Equipe -->
		<div
			v-if="isModalOpen"
			class="modal-backdrop"
			@click.self="closeModal"
		>
			<div
				class="modal-card"
				role="dialog"
				aria-modal="true"
			>
				<div class="modal-header">
					<h3 class="modal-title">
						{{ editingId ? 'Edit Team' : 'Create a Team' }}
					</h3>
					<button
						type="button"
						class="modal-close"
						@click="closeModal"
					>
						&times;
					</button>
				</div>
				<div class="modal-body">
					<div class="form-group">
						<label
							class="form-label"
							for="team-name-input"
						>Team Name *</label>
						<input
							id="team-name-input"
							v-model="modalName"
							type="text"
							class="form-input"
							placeholder="e.g. Engineering, Marketing"
							@keydown.enter="saveTeam"
						>
					</div>
					<div class="form-group">
						<label
							class="form-label"
							for="team-desc-input"
						>Description</label>
						<input
							id="team-desc-input"
							v-model="modalDescription"
							type="text"
							class="form-input"
							placeholder="What is this team responsible for?"
							@keydown.enter="saveTeam"
						>
					</div>
				</div>
				<div class="modal-footer">
					<button
						type="button"
						class="btn btn-secondary"
						@click="closeModal"
					>
						Cancel
					</button>
					<button
						type="button"
						class="btn btn-primary"
						@click="saveTeam"
					>
						{{ editingId ? 'Save Changes' : 'Create Team' }}
					</button>
				</div>
			</div>
		</div>

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

export interface TeamItem {
	id: number
	name: string
	description: string
	memberCount: number
}

const router = useRouter()
const authStore = useAuthStore()

const userInitials = computed(() => {
	const name = authStore.userDisplayName || 'Matheus'
	return name.substring(0, 2).toUpperCase()
})

const initialData: TeamItem[] = [
	{id: 1, name: 'Frontend Core', description: 'Web application & design system engineering', memberCount: 5},
	{id: 2, name: 'Design System', description: 'UI tokens, accessibility and visual components', memberCount: 3},
	{id: 3, name: 'Operations', description: 'Infrastructure, deployment and quality reliability', memberCount: 4},
]

const teams = ref<TeamItem[]>(JSON.parse(JSON.stringify(initialData)))
const isSimulatedEmpty = ref(false)

const isModalOpen = ref(false)
const editingId = ref<number | null>(null)
const modalName = ref('')
const modalDescription = ref('')

const toastMessage = ref('')
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(msg: string) {
	toastMessage.value = msg
	if (toastTimer) clearTimeout(toastTimer)
	toastTimer = setTimeout(() => {
		toastMessage.value = ''
	}, 3000)
}

const displayedTeams = computed(() => {
	if (isSimulatedEmpty.value) return []
	return teams.value
})

function onRowClick(team: TeamItem) {
	showToast(`Acessando equipe: ${team.name}`)
}

function openModal(id: number | null = null) {
	editingId.value = id
	if (id) {
		const team = teams.value.find(t => t.id === id)
		if (team) {
			modalName.value = team.name
			modalDescription.value = team.description
		}
	} else {
		modalName.value = ''
		modalDescription.value = ''
	}
	isModalOpen.value = true
}

function closeModal() {
	isModalOpen.value = false
	editingId.value = null
}

function saveTeam() {
	const name = modalName.value.trim()
	if (!name) {
		showToast('O nome da equipe é obrigatório')
		return
	}

	if (editingId.value) {
		const team = teams.value.find(t => t.id === editingId.value)
		if (team) {
			team.name = name
			team.description = modalDescription.value.trim()
			showToast('Equipe atualizada com sucesso')
		}
	} else {
		teams.value.unshift({
			id: Date.now(),
			name,
			description: modalDescription.value.trim(),
			memberCount: 1,
		})
		showToast('Equipe criada com sucesso')
	}

	closeModal()
}

function deleteTeam(id: number) {
	teams.value = teams.value.filter(t => t.id !== id)
	showToast('Equipe excluída')
}

function toggleEmptyMode() {
	isSimulatedEmpty.value = !isSimulatedEmpty.value
	showToast(isSimulatedEmpty.value ? 'Visualizando Estado Vazio' : 'Visualizando Lista Preenchida')
}

function resetData() {
	teams.value = JSON.parse(JSON.stringify(initialData))
	isSimulatedEmpty.value = false
	showToast('Dados restaurados')
}
</script>

<style scoped lang="scss">
.teams-page {
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
	--border-subtle: #f0f0f0;
	--border-focus: #5e6ad2;

	--accent-base: #5e6ad2;
	--accent-hover: #4a55b5;

	--shadow-card: 0 1px 3px rgba(0, 0, 0, 0.02), 0 1px 2px rgba(0, 0, 0, 0.04);
	--shadow-modal: 0 20px 40px rgba(0, 0, 0, 0.15);

	font-family: var(--font-family);
	background-color: var(--bg-app);
	color: var(--text-primary);
	min-height: 100vh;
	display: flex;
	flex-direction: column;
	width: 100%;
}

:global(html.dark) .teams-page {
	/* Design Tokens - Dark Theme */
	--bg-app: #0f0f0f;
	--bg-sidebar: #0f0f0f;
	--bg-card: #1c1c1c;
	--bg-hover: #2a2a2a;

	--text-primary: #e5e5e5;
	--text-secondary: #8a8a8a;
	--text-tertiary: #555555;

	--border-color: #333333;
	--border-subtle: #222222;

	--shadow-modal: 0 20px 40px rgba(0, 0, 0, 0.5);
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
	justify-content: center;
}

.teams-wrapper {
	width: 100%;
	max-width: 1000px;
	display: flex;
	flex-direction: column;
}

.view-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 24px;
}

.view-title {
	font-size: 24px;
	font-weight: 600;
	letter-spacing: -0.5px;
	color: var(--text-primary);
}

.btn-primary {
	background: var(--accent-base);
	color: #ffffff;
	border: 1px solid var(--accent-base);
	border-radius: 6px;
	padding: 6px 14px;
	font-size: 12px;
	font-weight: 600;
	cursor: pointer;
	display: flex;
	align-items: center;
	gap: 6px;
	transition: all 0.15s ease;

	&:hover {
		background: var(--accent-hover);
		border-color: var(--accent-hover);
	}
}

/* Table */
.teams-table {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 8px;
	box-shadow: var(--shadow-card);
	display: flex;
	flex-direction: column;
	overflow: hidden;
}

.team-row {
	display: grid;
	grid-template-columns: minmax(200px, 1fr) 250px 100px 60px;
	align-items: center;
	padding: 12px 16px;
	gap: 16px;
	border-bottom: 1px solid var(--border-subtle);
	cursor: pointer;
	transition: background 0.1s;

	&:last-child {
		border-bottom: none;
	}

	&:hover {
		background: var(--bg-hover);

		.action-btn {
			opacity: 1;
		}
	}
}

.cell-name {
	font-size: 14px;
	font-weight: 500;
	color: var(--text-primary);
	display: flex;
	align-items: center;
	gap: 10px;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.team-icon {
	color: var(--text-tertiary);
	flex-shrink: 0;
}

.name-text {
	overflow: hidden;
	text-overflow: ellipsis;
}

.cell-desc {
	font-size: 13px;
	color: var(--text-secondary);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.cell-members {
	font-size: 13px;
	color: var(--text-secondary);
	display: flex;
	align-items: center;
	gap: 6px;

	svg {
		color: var(--text-tertiary);
	}
}

.cell-actions {
	display: flex;
	align-items: center;
	justify-content: flex-end;
	gap: 6px;
}

.action-btn {
	background: transparent;
	border: none;
	cursor: pointer;
	color: var(--text-secondary);
	padding: 4px;
	border-radius: 4px;
	display: flex;
	align-items: center;
	justify-content: center;
	opacity: 0;
	transition: all 0.15s ease;

	&:hover {
		background: var(--bg-card);
		color: var(--text-primary);
	}

	&.danger:hover {
		color: #e5484d;
	}
}

/* Empty State */
.empty-state-view {
	display: flex;
	justify-content: center;
	align-items: center;
	padding: 64px 24px;
}

.empty-text {
	font-size: 14px;
	font-style: italic;
	color: var(--text-secondary);
}

.empty-link {
	color: var(--accent-base);
	font-style: normal;
	font-weight: 500;
	margin-left: 4px;
	cursor: pointer;

	&:hover {
		text-decoration: underline;
	}
}

/* Modal */
.modal-backdrop {
	position: fixed;
	top: 0;
	left: 0;
	width: 100vw;
	height: 100vh;
	background: rgba(0, 0, 0, 0.4);
	backdrop-filter: blur(2px);
	display: flex;
	align-items: center;
	justify-content: center;
	z-index: 1000;
}

.modal-card {
	width: 100%;
	max-width: 480px;
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 12px;
	box-shadow: var(--shadow-modal);
	display: flex;
	flex-direction: column;
	overflow: hidden;
}

.modal-header {
	padding: 16px 20px;
	border-bottom: 1px solid var(--border-subtle);
	display: flex;
	justify-content: space-between;
	align-items: center;
}

.modal-title {
	font-size: 16px;
	font-weight: 600;
	color: var(--text-primary);
}

.modal-close {
	background: transparent;
	border: none;
	font-size: 20px;
	cursor: pointer;
	color: var(--text-secondary);
	padding: 0 4px;

	&:hover {
		color: var(--text-primary);
	}
}

.modal-body {
	padding: 20px;
	display: flex;
	flex-direction: column;
	gap: 16px;
}

.form-group {
	display: flex;
	flex-direction: column;
	gap: 6px;
}

.form-label {
	font-size: 12px;
	font-weight: 600;
	color: var(--text-secondary);
	text-transform: uppercase;
	letter-spacing: 0.4px;
}

.form-input {
	background: var(--bg-app);
	border: 1px solid var(--border-color);
	border-radius: 6px;
	padding: 8px 12px;
	color: var(--text-primary);
	font-family: inherit;
	font-size: 14px;
	outline: none;
	transition: all 0.15s ease;

	&:focus {
		border-color: var(--border-focus);
	}
}

.modal-footer {
	padding: 14px 20px;
	border-top: 1px solid var(--border-subtle);
	display: flex;
	justify-content: flex-end;
	gap: 10px;
	background: var(--bg-card);
}

.btn {
	padding: 7px 14px;
	border-radius: 6px;
	font-size: 13px;
	font-weight: 600;
	cursor: pointer;
	border: none;
	transition: all 0.15s ease;
}

.btn-secondary {
	background: var(--bg-hover);
	color: var(--text-secondary);
	border: 1px solid var(--border-color);

	&:hover {
		background: var(--bg-card);
		color: var(--text-primary);
	}
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
	box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
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
