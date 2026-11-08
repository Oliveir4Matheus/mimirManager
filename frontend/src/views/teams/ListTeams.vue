<template>
	<div class="teams-page">
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
						@click="navigateToCreateTeam"
					>
						<svg
							width="14"
							height="14"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
						>
							<line x1="12" y1="5" x2="12" y2="19" />
							<line x1="5" y1="12" x2="19" y2="12" />
						</svg>
						Create a Team
					</button>
				</div>

				<!-- Teams Table Container -->
				<div
					v-if="teams.length > 0"
					class="teams-table"
				>
					<div
						v-for="team in teams"
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
								<circle cx="9" cy="7" r="4" />
								<path d="M23 21v-2a4 4 0 0 0-3-3.87" />
								<path d="M16 3.13a4 4 0 0 1 0 7.75" />
							</svg>
							<span class="name-text">{{ team.name }}</span>
						</div>

						<!-- Description -->
						<div class="cell-desc">
							{{ team.description || '-' }}
						</div>

						<!-- Members Count (empty cell placeholder) -->
						<div class="cell-members" />

						<!-- Actions -->
						<div
							class="cell-actions"
							@click.stop
						>
							<button
								type="button"
								class="action-btn"
								title="Editar Equipe"
								:aria-label="'Editar equipe ' + team.name"
								@click.stop="team.id && editTeam(team.id)"
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
								title="Excluir Equipe"
								:aria-label="'Excluir equipe ' + team.name"
								@click.stop="confirmDelete(team)"
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
					v-else-if="!isFetching"
					class="empty-state-view"
				>
					<div class="empty-icon-circle">
						<svg
							width="32"
							height="32"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
						>
							<path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2" />
							<circle cx="9" cy="7" r="4" />
							<path d="M23 21v-2a4 4 0 0 0-3-3.87" />
							<path d="M16 3.13a4 4 0 0 1 0 7.75" />
						</svg>
					</div>
					<h3 class="empty-title">
						Nenhuma equipe encontrada
					</h3>
					<p class="empty-text">
						Crie uma equipe para compartilhar projetos e colaborar em tarefas.
					</p>
					<button
						type="button"
						class="btn-primary"
						@click="navigateToCreateTeam"
					>
						Criar Equipe
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
	</div>
</template>

<script setup lang="ts">
import {ref} from 'vue'
import {useRouter} from 'vue-router'
import {useRouteQuery} from '@vueuse/router'
import {useTeamsPage} from '@/composables/useTeams'
import {useDeleteTeamMutation} from '@/client/queries/teams'
import {normalizePageNumber} from '@/client/queries/pagination'
import type {Team} from '@/client/generated'

const router = useRouter()
const page = useRouteQuery('page', '1', {transform: normalizePageNumber})
const {teams, isFetching} = useTeamsPage(page)
const deleteTeamMutation = useDeleteTeamMutation()

const toastMessage = ref('')
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(msg: string) {
	toastMessage.value = msg
	if (toastTimer) clearTimeout(toastTimer)
	toastTimer = setTimeout(() => {
		toastMessage.value = ''
	}, 3000)
}

function onRowClick(team: Team) {
	router.push({name: 'teams.edit', params: {id: team.id}})
}

function editTeam(id: number) {
	router.push({name: 'teams.edit', params: {id}})
}

function navigateToCreateTeam() {
	router.push({name: 'teams.new'})
}

async function confirmDelete(team: Team) {
	if (window.confirm(`Deseja realmente excluir a equipe "${team.name}"?`)) {
		try {
			if (team.id) await deleteTeamMutation.mutateAsync(team.id)
			showToast('Equipe excluída com sucesso')
		} catch (e) {
			showToast('Erro ao excluir equipe')
		}
	}
}
</script>

<style scoped lang="scss">
.teams-page {
	width: 100%;
}

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
	transition: all 0.15s;

	&:hover {
		background: var(--accent-hover);
		border-color: var(--accent-hover);
	}
}

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
	grid-template-columns: minmax(200px, 1.5fr) minmax(250px, 2fr) 80px 80px;
	gap: 16px;
	align-items: center;
	padding: 14px 16px;
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
	display: flex;
	align-items: center;
	gap: 10px;
	font-size: 14px;
	font-weight: 500;
	color: var(--text-primary);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.team-icon {
	color: var(--text-tertiary);
	flex-shrink: 0;
}

.cell-desc {
	font-size: 13px;
	color: var(--text-secondary);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.cell-members {
	font-size: 12px;
	color: var(--text-tertiary);
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
	display: flex;
	align-items: center;
	justify-content: center;
	padding: 4px;
	border-radius: 4px;
	opacity: 0;
	transition: all 0.15s;

	&:hover {
		background: var(--bg-card);
		color: var(--text-primary);
	}

	&.danger:hover {
		color: var(--priority-urgent);
	}
}

.empty-state-view {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 64px 24px;
	text-align: center;
}

.empty-icon-circle {
	width: 64px;
	height: 64px;
	background: var(--bg-hover);
	color: var(--text-tertiary);
	border-radius: 50%;
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
	font-size: 13px;
	color: var(--text-secondary);
	margin-bottom: 20px;
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
</style>
