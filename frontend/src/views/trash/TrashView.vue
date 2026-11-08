<template>
	<div class="trash-page">
		<!-- Content Area -->
		<main class="content-area">
			<div class="trash-wrapper">
				<!-- Header Section -->
				<div class="view-header">
					<h1 class="view-title">
						Trash
					</h1>
					<button
						type="button"
						class="btn-danger"
						:disabled="currentItems.length === 0"
						@click="emptyTrash"
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
						Empty Trash
					</button>
				</div>

				<!-- Tabs (Segmented Control) -->
				<div
					class="tabs-container"
					role="tablist"
				>
					<button
						type="button"
						role="tab"
						class="tab-btn"
						:class="{active: activeTab === 'tasks'}"
						:aria-selected="activeTab === 'tasks'"
						@click="activeTab = 'tasks'"
					>
						Tasks
					</button>
					<button
						type="button"
						role="tab"
						class="tab-btn"
						:class="{active: activeTab === 'projects'}"
						:aria-selected="activeTab === 'projects'"
						@click="activeTab = 'projects'"
					>
						Projects
					</button>
				</div>

				<!-- Table Container -->
				<div
					v-if="currentItems.length > 0"
					class="table-container"
				>
					<!-- Table Header -->
					<div
						class="table-header-row"
						:class="{'projects-mode': activeTab === 'projects'}"
					>
						<div>{{ activeTab === 'tasks' ? 'Tarefa' : 'Projeto' }}</div>
						<div v-if="activeTab === 'tasks'">
							Projeto
						</div>
						<div>Excluído em</div>
						<div><!-- Actions column --></div>
					</div>

					<!-- Table Rows -->
					<div
						v-for="item in currentItems"
						:key="item.id"
						class="grid-row"
						:class="{'projects-mode': activeTab === 'projects'}"
					>
						<!-- Title -->
						<div class="cell-title">
							{{ item.title }}
						</div>

						<!-- Project Pill (Tasks only) -->
						<div
							v-if="activeTab === 'tasks'"
							class="cell-project"
						>
							<span class="project-pill">
								<span
									class="project-dot"
									:style="{backgroundColor: getProjectColor(item.projectId)}"
								/>
								{{ getProjectTitle(item.projectId) }}
							</span>
						</div>

						<!-- Deleted At -->
						<div class="cell-date">
							{{ formatDateSince(item.deletedAt) }}
						</div>

						<!-- Row Actions -->
						<div class="cell-actions">
							<button
								type="button"
								class="action-btn"
								title="Restaurar"
								:aria-label="'Restaurar ' + item.title"
								@click="restoreItem(item.id)"
							>
								Restore
							</button>
							<button
								type="button"
								class="action-btn danger"
								title="Excluir Permanentemente"
								:aria-label="'Excluir permanentemente ' + item.title"
								@click="deletePermanently(item.id)"
							>
								Delete
							</button>
						</div>
					</div>
				</div>

				<!-- Empty State View -->
				<div
					v-else
					class="empty-state-view"
				>
					<p class="empty-text">
						{{ activeTab === 'tasks' ? 'There are no tasks in the trash.' : 'There are no projects in the trash.' }}
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
	</div>
</template>

<script setup lang="ts">
import {computed, ref} from 'vue'
import {useQuery} from '@tanstack/vue-query'
import {useProjects} from '@/composables/useProjects'
import {
	trashedTasksQuery,
	trashedProjectsQuery,
	useRestoreTaskMutation,
	useDeleteTaskPermanentlyMutation,
	useEmptyTaskTrashMutation,
	useRestoreProjectMutation,
	useDeleteProjectPermanentlyMutation,
	useEmptyProjectTrashMutation,
} from '@/client/queries/trash'
import {formatDateSince} from '@/helpers/time/formatDate'

export interface TrashDisplayItem {
	id: number
	title: string
	projectId?: number
	deletedAt?: string | null
}

const activeTab = ref<'tasks' | 'projects'>('tasks')
const currentPage = ref(1)

const taskQuery = useQuery(computed(() => trashedTasksQuery(currentPage.value)))
const projectQuery = useQuery(computed(() => trashedProjectsQuery(currentPage.value)))

const restoreTaskMutation = useRestoreTaskMutation()
const deleteTaskPermanentlyMutation = useDeleteTaskPermanentlyMutation()
const emptyTaskTrashMutation = useEmptyTaskTrashMutation()

const restoreProjectMutation = useRestoreProjectMutation()
const deleteProjectPermanentlyMutation = useDeleteProjectPermanentlyMutation()
const emptyProjectTrashMutation = useEmptyProjectTrashMutation()

const projectList = useProjects()

const toastMessage = ref('')
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(msg: string) {
	toastMessage.value = msg
	if (toastTimer) clearTimeout(toastTimer)
	toastTimer = setTimeout(() => {
		toastMessage.value = ''
	}, 3000)
}

const currentItems = computed<TrashDisplayItem[]>(() => {
	if (activeTab.value === 'tasks') {
		const list = taskQuery.data.value?.items ?? []
		return list.map(t => ({
			id: t.id,
			title: t.title,
			projectId: t.project_id,
			deletedAt: t.deleted_at,
		}))
	}
	const list = projectQuery.data.value?.items ?? []
	return list.map(p => ({
		id: p.id,
		title: p.title,
		deletedAt: p.deleted_at,
	}))
})

function getProjectColor(projectId?: number): string {
	if (!projectId) return '#9ca3af'
	const color = projectList.projects[projectId]?.hex_color
	return color ? '#' + color : '#9ca3af'
}

function getProjectTitle(projectId?: number): string {
	if (!projectId) return 'Inbox'
	return projectList.projects[projectId]?.title || 'Inbox'
}

async function restoreItem(id: number) {
	try {
		if (activeTab.value === 'tasks') {
			await restoreTaskMutation.mutateAsync(id)
			showToast('Tarefa restaurada com sucesso')
		} else {
			await restoreProjectMutation.mutateAsync(id)
			showToast('Projeto restaurado com sucesso')
		}
	} catch (e) {
		showToast('Erro ao restaurar item')
	}
}

async function deletePermanently(id: number) {
	try {
		if (activeTab.value === 'tasks') {
			await deleteTaskPermanentlyMutation.mutateAsync(id)
			showToast('Tarefa excluída permanentemente')
		} else {
			await deleteProjectPermanentlyMutation.mutateAsync(id)
			showToast('Projeto excluído permanentemente')
		}
	} catch (e) {
		showToast('Erro ao excluir item')
	}
}

async function emptyTrash() {
	try {
		if (activeTab.value === 'tasks') {
			await emptyTaskTrashMutation.mutateAsync({includeOthers: false})
			showToast('Lixeira de tarefas esvaziada')
		} else {
			await emptyProjectTrashMutation.mutateAsync({includeOthers: false})
			showToast('Lixeira de projetos esvaziada')
		}
	} catch (e) {
		showToast('Erro ao esvaziar lixeira')
	}
}
</script>

<style scoped lang="scss">
.trash-page {
	/* Design Tokens - Light Theme */
	--font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
	--bg-app: #f9f9f9;
	--bg-sidebar: #fcfcfc;
	--bg-card: #ffffff;
	--bg-hover: #f4f4f5;
	--bg-header-row: #fafafa;
	--bg-tabs: #f0f0f0;

	--text-primary: #111111;
	--text-secondary: #6b6b6b;
	--text-tertiary: #9ca3af;

	--border-color: #e5e5e5;
	--border-subtle: #f0f0f0;
	--border-focus: #5e6ad2;

	--accent-base: #5e6ad2;
	--accent-hover: #4a55b5;
	--danger-base: #e5484d;
	--danger-hover: #cc3a3e;
	--danger-bg: rgba(229, 72, 77, 0.1);

	--shadow-card: 0 1px 3px rgba(0, 0, 0, 0.02), 0 1px 2px rgba(0, 0, 0, 0.04);

	font-family: var(--font-family);
	background-color: var(--bg-app);
	color: var(--text-primary);
	min-height: 100vh;
	display: flex;
	flex-direction: column;
	width: 100%;
}

:global(html.dark) .trash-page {
	/* Design Tokens - Dark Theme */
	--bg-app: #0f0f0f;
	--bg-sidebar: #0f0f0f;
	--bg-card: #1c1c1c;
	--bg-hover: #2a2a2a;
	--bg-header-row: #181b22;
	--bg-tabs: #222222;

	--text-primary: #e5e5e5;
	--text-secondary: #8a8a8a;
	--text-tertiary: #555555;

	--border-color: #333333;
	--border-subtle: #222222;
	--danger-base: #ff6369;
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

.trash-wrapper {
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

.btn-danger {
	display: flex;
	align-items: center;
	gap: 6px;
	background: transparent;
	border: 1px solid var(--danger-base);
	color: var(--danger-base);
	padding: 6px 12px;
	border-radius: 6px;
	font-size: 12px;
	font-weight: 600;
	text-transform: uppercase;
	letter-spacing: 0.5px;
	cursor: pointer;
	transition: all 0.15s ease;

	&:hover {
		background: var(--danger-bg);
	}
}

/* Tabs */
.tabs-container {
	display: flex;
	background: var(--bg-tabs);
	padding: 2px;
	border-radius: 6px;
	width: fit-content;
	margin-bottom: 16px;
}

.tab-btn {
	background: transparent;
	border: none;
	padding: 4px 12px;
	font-size: 13px;
	font-weight: 500;
	color: var(--text-secondary);
	border-radius: 4px;
	cursor: pointer;
	transition: all 0.15s ease;

	&.active {
		background: var(--bg-card);
		color: var(--text-primary);
		font-weight: 600;
		box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
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
	overflow: hidden;
}

.table-header-row,
.grid-row {
	display: grid;
	grid-template-columns: minmax(200px, 1fr) 150px 120px 140px;
	align-items: center;
	padding: 8px 16px;
	gap: 16px;

	&.projects-mode {
		grid-template-columns: minmax(200px, 1fr) 120px 140px;
	}
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
	}
}

.cell-title {
	font-size: 13px;
	font-weight: 500;
	color: var(--text-primary);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
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

.cell-actions {
	display: flex;
	align-items: center;
	justify-content: flex-end;
	gap: 8px;
}

.action-btn {
	background: transparent;
	border: none;
	font-size: 12px;
	font-weight: 500;
	cursor: pointer;
	color: var(--text-secondary);
	padding: 3px 6px;
	border-radius: 4px;
	opacity: 0;
	transition: all 0.15s ease;

	&:hover {
		background: var(--bg-hover);
		color: var(--text-primary);
	}

	&.danger:hover {
		color: var(--danger-base);
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
