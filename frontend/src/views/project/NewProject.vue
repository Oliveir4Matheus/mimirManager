<template>
	<div class="create-project-page">
		<!-- Header Superior -->
		<header class="header">
			<div class="breadcrumb">
				<RouterLink
					:to="{name: 'projects.index'}"
					class="breadcrumb-link"
				>
					Projetos
				</RouterLink>
				<span class="breadcrumb-separator">/</span>
				<span class="breadcrumb-current">Novo Projeto</span>
			</div>
		</header>

		<!-- Corpo de Conteúdo -->
		<main class="content-body">
			<div class="card">
				<h1 class="card-title">
					Criar Novo Projeto
				</h1>
				<p class="card-subtitle">
					Organize suas tarefas, defina visões e colabore com a sua equipe.
				</p>

				<form @submit.prevent="createProject">
					<!-- Título -->
					<div class="form-group">
						<label
							class="form-label"
							for="proj-title"
						>Título do Projeto *</label>
						<input
							id="proj-title"
							v-model="projectTitle"
							type="text"
							class="form-input"
							:class="{'is-invalid': showError && !projectTitle}"
							placeholder="Ex: Rebrand do Sistema 2026"
							required
							autofocus
						>
						<span
							v-if="showError && !projectTitle"
							class="error-text"
						>
							O título do projeto é obrigatório.
						</span>
					</div>

					<!-- Descrição -->
					<div class="form-group">
						<label
							class="form-label"
							for="proj-desc"
						>Descrição (Opcional)</label>
						<textarea
							id="proj-desc"
							v-model="projectDescription"
							class="form-textarea"
							placeholder="Objetivos principais, links de referência e escopo..."
						/>
					</div>

					<!-- Projeto Pai -->
					<div class="form-group">
						<label
							class="form-label"
							for="proj-parent"
						>Projeto Pai</label>
						<select
							id="proj-parent"
							v-model="parentProjectId"
							class="form-select"
						>
							<option :value="null">
								Nenhum (Projeto Raiz)
							</option>
							<option
								v-for="p in existingProjects"
								:key="p.id"
								:value="p.id"
							>
								{{ p.title }}
							</option>
						</select>
						<span class="form-hint">Selecione caso este projeto pertença a uma hierarquia existente.</span>
					</div>

					<!-- Cor de Identificação -->
					<div class="form-group">
						<label
							class="form-label"
							for="proj-color"
						>Cor de Identificação</label>
						<div class="color-picker-row">
							<input
								id="proj-color"
								v-model="selectedColor"
								type="color"
								class="color-input"
							>
							<span class="color-label">{{ selectedColor.toUpperCase() }}</span>
						</div>
					</div>

					<!-- Botões de Ação -->
					<div class="btn-group">
						<button
							type="button"
							class="btn btn-secondary"
							@click="cancel"
						>
							Cancelar
						</button>
						<button
							type="submit"
							class="btn btn-primary"
							:disabled="isSubmitting"
						>
							<span v-if="isSubmitting">Criando...</span>
							<span v-else>Criar Projeto</span>
						</button>
					</div>
				</form>
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
import {useCreateProjectMutation} from '@/client/queries/projects'

const router = useRouter()
const projectList = useProjects()
const createProjectMutation = useCreateProjectMutation()

const projectTitle = ref('')
const projectDescription = ref('')
const parentProjectId = ref<number | null>(null)
const selectedColor = ref('#5e6ad2')
const showError = ref(false)
const isSubmitting = ref(false)

const toastMessage = ref('')
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(msg: string) {
	toastMessage.value = msg
	if (toastTimer) clearTimeout(toastTimer)
	toastTimer = setTimeout(() => {
		toastMessage.value = ''
	}, 3000)
}

const existingProjects = computed(() => projectList.projectsArray ?? [])

function cancel() {
	router.push({name: 'projects.index'})
}

async function createProject() {
	if (!projectTitle.value.trim()) {
		showError.value = true
		return
	}
	showError.value = false
	isSubmitting.value = true

	try {
		const created = await createProjectMutation.mutateAsync({
			title: projectTitle.value.trim(),
			description: projectDescription.value.trim() || undefined,
			parent_project_id: parentProjectId.value || 0,
			hex_color: selectedColor.value.replace('#', '') || undefined,
		})
		showToast(`Projeto "${projectTitle.value}" criado com sucesso!`)
		router.push({name: 'project.view', params: {id: created.id}})
	} catch (e) {
		showToast('Erro ao criar projeto')
	} finally {
		isSubmitting.value = false
	}
}
</script>

<style scoped lang="scss">
.create-project-page {
	/* Design Tokens - Light Theme */
	--font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
	--bg-app: #f9f9f9;
	--bg-sidebar: #fcfcfc;
	--bg-card: #ffffff;
	--bg-input: #f1f3f7;
	--bg-hover: #f4f4f5;

	--text-primary: #111111;
	--text-secondary: #6b6b6b;
	--text-tertiary: #9ca3af;

	--border-color: #e5e5e5;
	--border-subtle: #f0f0f0;
	--border-focus: #5e6ad2;

	--accent-base: #5e6ad2;
	--accent-hover: #6e7be0;
	--accent-gradient: linear-gradient(135deg, #5e6ad2 0%, #8b5cf6 100%);

	--radius-sm: 6px;
	--radius-md: 8px;
	--radius-lg: 12px;
	--radius-xl: 16px;

	--shadow-card: 0 4px 20px rgba(0, 0, 0, 0.05);

	font-family: var(--font-family);
	background-color: var(--bg-app);
	color: var(--text-primary);
	min-height: 100vh;
	display: flex;
	flex-direction: column;
	width: 100%;
}

:global(html.dark) .create-project-page {
	/* Design Tokens - Dark Theme */
	--bg-app: #0d0e12;
	--bg-sidebar: #12141a;
	--bg-card: #181b22;
	--bg-input: #1f232d;
	--bg-hover: #262a36;

	--text-primary: #f0f2f5;
	--text-secondary: #9aa1b2;
	--text-tertiary: #656c80;

	--border-color: #232836;
	--border-subtle: #1c202c;
	--border-focus: #5e6ad2;

	--shadow-card: 0 4px 20px rgba(0, 0, 0, 0.25);
}

/* Header */
.header {
	height: 56px;
	border-bottom: 1px solid var(--border-color);
	display: flex;
	align-items: center;
	padding: 0 32px;
	background: var(--bg-app);
	position: sticky;
	top: 0;
	z-index: 10;
}

.breadcrumb {
	display: flex;
	align-items: center;
	gap: 8px;
	font-size: 14px;
	font-weight: 600;

	.breadcrumb-link {
		color: var(--text-secondary);
		text-decoration: none;

		&:hover {
			color: var(--text-primary);
		}
	}

	.breadcrumb-separator {
		color: var(--text-tertiary);
		font-weight: 400;
	}

	.breadcrumb-current {
		color: var(--text-primary);
	}
}

/* Content Body */
.content-body {
	padding: 40px 32px;
	max-width: 900px;
	width: 100%;
	margin: 0 auto;
}

.card {
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: var(--radius-lg);
	padding: 28px;
	box-shadow: var(--shadow-card);
}

.card-title {
	font-size: 18px;
	font-weight: 600;
	letter-spacing: -0.4px;
	margin-bottom: 6px;
	color: var(--text-primary);
}

.card-subtitle {
	font-size: 13px;
	color: var(--text-secondary);
	margin-bottom: 24px;
}

.form-group {
	display: flex;
	flex-direction: column;
	gap: 6px;
	margin-bottom: 20px;
}

.form-label {
	font-size: 12px;
	font-weight: 600;
	color: var(--text-secondary);
	text-transform: uppercase;
	letter-spacing: 0.4px;
}

.form-input,
.form-textarea,
.form-select {
	background: var(--bg-input);
	border: 1px solid var(--border-color);
	border-radius: var(--radius-md);
	padding: 10px 14px;
	color: var(--text-primary);
	font-family: inherit;
	font-size: 14px;
	outline: none;
	transition: all 0.15s ease;

	&:focus {
		border-color: var(--border-focus);
		box-shadow: 0 0 0 2px rgba(94, 106, 210, 0.2);
	}

	&.is-invalid {
		border-color: #ef4444;
	}
}

.error-text {
	color: #ef4444;
	font-size: 12px;
	margin-top: 2px;
}

.form-textarea {
	resize: vertical;
	min-height: 80px;
}

.form-hint {
	font-size: 12px;
	color: var(--text-tertiary);
	margin-top: 4px;
}

.color-picker-row {
	display: flex;
	gap: 10px;
	align-items: center;
	margin-top: 6px;
}

.color-input {
	width: 38px;
	height: 38px;
	border-radius: 6px;
	border: 1px solid var(--border-color);
	background: var(--bg-input);
	cursor: pointer;
	padding: 0;
}

.color-label {
	font-size: 13px;
	color: var(--text-secondary);
	font-weight: 500;
}

/* Buttons */
.btn-group {
	display: flex;
	align-items: center;
	justify-content: flex-end;
	gap: 12px;
	margin-top: 28px;
	padding-top: 20px;
	border-top: 1px solid var(--border-subtle);
}

.btn {
	display: inline-flex;
	align-items: center;
	justify-content: center;
	gap: 8px;
	padding: 9px 18px;
	border-radius: var(--radius-md);
	font-size: 13px;
	font-weight: 600;
	cursor: pointer;
	transition: all 0.15s ease;
	border: none;
	font-family: inherit;
}

.btn-primary {
	background: var(--accent-base);
	color: #ffffff;
	box-shadow: 0 2px 8px rgba(94, 106, 210, 0.35);

	&:hover:not(:disabled) {
		background: var(--accent-hover);
		transform: translateY(-1px);
	}

	&:disabled {
		opacity: 0.6;
		cursor: not-allowed;
	}
}

.btn-secondary {
	background: var(--bg-input);
	color: var(--text-secondary);
	border: 1px solid var(--border-color);

	&:hover {
		background: var(--bg-hover);
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
</style>
