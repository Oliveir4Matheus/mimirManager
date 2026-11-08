<template>
	<div class="labels-page">
		<!-- Content Area -->
		<main class="content-area">
			<div class="labels-wrapper">
				<!-- Header Section -->
				<div class="view-header">
					<h1 class="view-title">
						Manage labels
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
							<line x1="12" y1="5" x2="12" y2="19" />
							<line x1="5" y1="12" x2="19" y2="12" />
						</svg>
						New Label
					</button>
				</div>

				<!-- Labels Table Container -->
				<div
					v-if="labels.length > 0"
					class="labels-table"
				>
					<div
						v-for="label in labels"
						:key="label.id"
						class="label-row"
						@click="openModal(label)"
					>
						<!-- Dot -->
						<div class="cell-dot">
							<div
								class="label-color-dot"
								:style="{backgroundColor: label.hex_color ? '#' + label.hex_color : '#9ca3af'}"
							/>
						</div>

						<!-- Name -->
						<div class="cell-name">
							{{ label.title }}
						</div>

						<!-- Description -->
						<div class="cell-desc">
							{{ label.description || '-' }}
						</div>

						<!-- Count (empty cell placeholder) -->
						<div class="cell-count" />

						<!-- Actions -->
						<div
							class="cell-actions"
							@click.stop
						>
							<button
								type="button"
								class="action-btn"
								title="Editar"
								:aria-label="'Editar ' + label.title"
								@click.stop="openModal(label)"
							>
								<svg
									width="14"
									height="14"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2"
								>
									<path d="M12 20h9" />
									<path d="M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z" />
								</svg>
							</button>
							<button
								type="button"
								class="action-btn danger"
								title="Excluir"
								:aria-label="'Excluir ' + label.title"
								@click.stop="confirmDelete(label)"
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
					v-else-if="!isPending"
					class="empty-state-view"
				>
					<p class="empty-text">
						You currently do not have any labels.
						<a
							class="empty-link"
							@click="openModal()"
						>Create a label.</a>
					</p>
				</div>
			</div>
		</main>

		<!-- Modal Criar / Editar Label -->
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
						{{ editingLabel ? 'Edit Label' : 'Create New Label' }}
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
							for="label-name-input"
						>Name *</label>
						<input
							id="label-name-input"
							v-model="modalName"
							type="text"
							class="form-input"
							placeholder="e.g. Bug, Feature, Design"
							@keydown.enter="saveLabel"
						>
					</div>
					<div class="form-group">
						<label
							class="form-label"
							for="label-desc-input"
						>Description</label>
						<input
							id="label-desc-input"
							v-model="modalDescription"
							type="text"
							class="form-input"
							placeholder="What is this label for?"
							@keydown.enter="saveLabel"
						>
					</div>
					<div class="form-group">
						<label class="form-label">Color</label>
						<div
							class="color-radios"
							role="radiogroup"
						>
							<label
								v-for="color in colorPalette"
								:key="color.hex"
								class="color-radio-label"
							>
								<input
									v-model="modalColor"
									type="radio"
									name="label-color"
									:value="color.hex.replace('#', '')"
									class="color-radio"
									:style="{backgroundColor: color.hex}"
									:aria-label="color.name"
								>
							</label>
						</div>
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
						:disabled="createMutation.isPending.value || updateMutation.isPending.value"
						@click="saveLabel"
					>
						Save Label
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
	</div>
</template>

<script setup lang="ts">
import {ref} from 'vue'
import {useLabels} from '@/composables/useLabels'
import {useCreateLabelMutation, useUpdateLabelMutation, useDeleteLabelMutation} from '@/client/queries/labels'
import type {Label} from '@/client/generated'

const {labels, isPending} = useLabels()
const createMutation = useCreateLabelMutation()
const updateMutation = useUpdateLabelMutation()
const deleteMutation = useDeleteLabelMutation()

const colorPalette = [
	{name: 'Vermelho', hex: '#e5484d'},
	{name: 'Laranja', hex: '#f5a623'},
	{name: 'Verde', hex: '#10b981'},
	{name: 'Roxo', hex: '#8b5cf6'},
	{name: 'Azul', hex: '#5e6ad2'},
	{name: 'Cinza', hex: '#9ca3af'},
]

const isModalOpen = ref(false)
const editingLabel = ref<Label | null>(null)
const modalName = ref('')
const modalDescription = ref('')
const modalColor = ref('e5484d')

const toastMessage = ref('')
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(msg: string) {
	toastMessage.value = msg
	if (toastTimer) clearTimeout(toastTimer)
	toastTimer = setTimeout(() => {
		toastMessage.value = ''
	}, 3000)
}

function openModal(label: Label | null = null) {
	editingLabel.value = label
	if (label) {
		modalName.value = label.title ?? ''
		modalDescription.value = label.description ?? ''
		modalColor.value = label.hex_color || 'e5484d'
	} else {
		modalName.value = ''
		modalDescription.value = ''
		modalColor.value = 'e5484d'
	}
	isModalOpen.value = true
}

function closeModal() {
	isModalOpen.value = false
	editingLabel.value = null
}

async function saveLabel() {
	const title = modalName.value.trim()
	if (!title) {
		showToast('O nome da etiqueta é obrigatório')
		return
	}

	try {
		if (editingLabel.value && editingLabel.value.id) {
			await updateMutation.mutateAsync({
				id: editingLabel.value.id,
				title,
				description: modalDescription.value.trim(),
				hex_color: modalColor.value.replace('#', ''),
			})
			showToast('Etiqueta atualizada com sucesso')
		} else {
			await createMutation.mutateAsync({
				title,
				description: modalDescription.value.trim(),
				hex_color: modalColor.value.replace('#', ''),
			})
			showToast('Etiqueta criada com sucesso')
		}
		closeModal()
	} catch (e) {
		showToast('Erro ao salvar etiqueta')
	}
}

async function confirmDelete(label: Label) {
	if (window.confirm(`Deseja realmente excluir a etiqueta "${label.title}"?`)) {
		try {
			await deleteMutation.mutateAsync(label)
			showToast('Etiqueta excluída com sucesso')
		} catch (e) {
			showToast('Erro ao excluir etiqueta')
		}
	}
}
</script>

<style scoped lang="scss">
.labels-page {
	width: 100%;
}

.content-area {
	flex: 1;
	padding: 32px 24px;
	overflow-y: auto;
	display: flex;
	justify-content: center;
}

.labels-wrapper {
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

.labels-table {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 8px;
	box-shadow: var(--shadow-card);
	display: flex;
	flex-direction: column;
	overflow: hidden;
}

.label-row {
	display: grid;
	grid-template-columns: 24px minmax(200px, 1fr) 200px 100px 60px;
	gap: 16px;
	align-items: center;
	padding: 12px 16px;
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

.cell-dot {
	display: flex;
	align-items: center;
}

.label-color-dot {
	width: 12px;
	height: 12px;
	border-radius: 50%;
}

.cell-name {
	font-size: 14px;
	font-weight: 500;
	color: var(--text-primary);
	white-space: nowrap;
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

.cell-count {
	font-size: 13px;
	color: var(--text-tertiary);
	text-align: right;
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
	justify-content: center;
	align-items: center;
	padding: 64px 24px;
}

.empty-text {
	font-size: 14px;
	color: var(--text-secondary);
	font-style: italic;
}

.empty-link {
	color: var(--accent-base);
	cursor: pointer;
	font-style: normal;
	font-weight: 500;
	margin-left: 4px;

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
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 12px;
	width: 100%;
	max-width: 480px;
	box-shadow: var(--shadow-modal);
	display: flex;
	flex-direction: column;
	overflow: hidden;
}

.modal-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 16px 20px;
	border-bottom: 1px solid var(--border-subtle);
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
	color: var(--text-secondary);
	cursor: pointer;
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
	font-size: 14px;
	color: var(--text-primary);
	outline: none;
	font-family: inherit;
	transition: all 0.15s;

	&:focus {
		border-color: var(--border-focus);
	}
}

.color-radios {
	display: flex;
	gap: 12px;
	align-items: center;
	margin-top: 4px;
}

.color-radio-label {
	display: flex;
	align-items: center;
	cursor: pointer;
}

.color-radio {
	appearance: none;
	width: 24px;
	height: 24px;
	border-radius: 50%;
	border: 2px solid transparent;
	outline: none;
	cursor: pointer;
	transition: all 0.15s;

	&:checked {
		border-color: var(--text-primary);
		transform: scale(1.15);
	}
}

.modal-footer {
	display: flex;
	justify-content: flex-end;
	gap: 10px;
	padding: 14px 20px;
	border-top: 1px solid var(--border-subtle);
	background: var(--bg-card);
}

.btn {
	padding: 7px 14px;
	border-radius: 6px;
	font-size: 13px;
	font-weight: 600;
	cursor: pointer;
	border: none;
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
