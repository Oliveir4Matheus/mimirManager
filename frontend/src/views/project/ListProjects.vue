<template>
	<div class="projects-page">
		<!-- Content Area -->
		<main class="content-area">
			<div class="projects-wrapper">
				<!-- Toolbar (Filtros e Ações) -->
				<div class="projects-toolbar">
					<label class="checkbox-label">
						<input
							v-model="showArchived"
							type="checkbox"
						>
						<span>Show Archived</span>
					</label>

					<div class="action-buttons">
						<button
							type="button"
							class="btn btn-secondary"
							@click="createSavedFilter"
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
							@click="createNewProject"
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
							New Project
						</button>
					</div>
				</div>

				<!-- Grid de Cards de Projetos (Populated) -->
				<div
					v-if="displayedProjects.length > 0"
					class="projects-grid"
				>
					<div
						v-for="project in displayedProjects"
						:key="project.id"
						class="project-card"
						tabindex="0"
						:aria-label="'Abrir projeto ' + getProjectTitle(project)"
						@click="openProject(project)"
						@keydown.enter="openProject(project)"
					>
						<!-- Cabeçalho do Card -->
						<div class="card-header">
							<div
								class="folder-icon"
								:style="{color: project.hex_color ? '#' + project.hex_color : 'var(--text-tertiary)'}"
							>
								<svg
									width="20"
									height="20"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2"
								>
									<path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z" />
								</svg>
							</div>
							<div
								v-if="project.is_archived"
								class="badge-archived"
							>
								Arquivado
							</div>
						</div>

						<!-- Título do Projeto -->
						<div class="project-title">
							{{ getProjectTitle(project) }}
						</div>
					</div>
				</div>

				<!-- Empty State -->
				<div
					v-else-if="!projectList.isLoading"
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
						@click="createNewProject"
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
	</div>
</template>

<script setup lang="ts">
import {computed, ref} from 'vue'
import {useRouter} from 'vue-router'
import {useProjects} from '@/composables/useProjects'
import {getProjectTitle} from '@/helpers/getProjectTitle'
import type {ProjectResponse} from '@/client/queries/projects'

const router = useRouter()
const projectList = useProjects()

const showArchived = ref(false)
const toastMessage = ref('')
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(msg: string) {
	toastMessage.value = msg
	if (toastTimer) clearTimeout(toastTimer)
	toastTimer = setTimeout(() => {
		toastMessage.value = ''
	}, 3000)
}

const displayedProjects = computed(() => {
	const all = projectList.projectsArray ?? []
	return showArchived.value ? all : all.filter((p: ProjectResponse) => !p.is_archived)
})

function openProject(project: ProjectResponse) {
	const viewId = project.views?.[0]?.id || 1
	router.push({
		name: 'project.view',
		params: {
			projectId: project.id,
			viewId,
		},
	})
}

function createNewProject() {
	router.push({name: 'project.create'})
}

function createSavedFilter() {
	router.push({name: 'filters.create'})
}
</script>

<style scoped lang="scss">
.projects-page {
	width: 100%;
}

.content-area {
	flex: 1;
	padding: 32px 24px;
	overflow-y: auto;
	display: flex;
	justify-content: center;
}

.projects-wrapper {
	width: 100%;
	max-width: 1000px;
	display: flex;
	flex-direction: column;
}

.projects-toolbar {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 24px;
}

.checkbox-label {
	display: flex;
	align-items: center;
	gap: 8px;
	font-size: 13px;
	color: var(--text-secondary);
	cursor: pointer;
	user-select: none;

	input[type="checkbox"] {
		cursor: pointer;
		accent-color: var(--accent-base);
	}
}

.action-buttons {
	display: flex;
	align-items: center;
	gap: 12px;
}

.btn {
	display: flex;
	align-items: center;
	gap: 6px;
	padding: 6px 14px;
	border-radius: 6px;
	font-size: 12px;
	font-weight: 600;
	cursor: pointer;
	transition: all 0.15s;
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

.btn-primary {
	background: var(--accent-base);
	color: #ffffff;
	border: 1px solid var(--accent-base);

	&:hover {
		background: var(--accent-hover);
		border-color: var(--accent-hover);
	}
}

.projects-grid {
	display: grid;
	grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
	gap: 16px;
}

.project-card {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 8px;
	padding: 16px;
	display: flex;
	flex-direction: column;
	justify-content: space-between;
	height: 110px;
	box-shadow: var(--shadow-card);
	cursor: pointer;
	transition: border-color 0.15s, transform 0.15s, box-shadow 0.15s;
	outline: none;

	&:hover,
	&:focus-visible {
		border-color: var(--accent-base);
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
	}
}

.card-header {
	display: flex;
	justify-content: space-between;
	align-items: flex-start;
}

.folder-icon {
	display: flex;
	align-items: center;
}

.badge-archived {
	font-size: 10px;
	font-weight: 500;
	color: var(--text-tertiary);
	background: var(--bg-hover);
	padding: 2px 6px;
	border-radius: 4px;
}

.project-title {
	font-size: 14px;
	font-weight: 500;
	color: var(--text-primary);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
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
