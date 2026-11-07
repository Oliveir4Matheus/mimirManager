<template>
	<div class="create-team-page">
		<!-- Header Superior -->
		<header class="header">
			<div class="breadcrumb">
				<RouterLink
					:to="{name: 'teams.index'}"
					class="breadcrumb-link"
				>
					Equipes
				</RouterLink>
				<span class="breadcrumb-separator">/</span>
				<span class="breadcrumb-current">Criar Equipe</span>
			</div>
		</header>

		<!-- Corpo de Conteúdo -->
		<main class="content-body">
			<div class="card">
				<h1 class="card-title">
					Criar Nova Equipe
				</h1>
				<p class="card-subtitle">
					Reúna membros para compartilhar permissões de projetos e tarefas centralizadas.
				</p>

				<form @submit.prevent="createTeam">
					<!-- Nome da Equipe -->
					<div class="form-group">
						<label
							class="form-label"
							for="team-name"
						>Nome da Equipe *</label>
						<input
							id="team-name"
							v-model="teamName"
							type="text"
							class="form-input"
							:class="{'is-invalid': showError && !teamName}"
							placeholder="Ex: Squad de Produto, Engenharia Frontend"
							required
							autofocus
						>
						<span
							v-if="showError && !teamName"
							class="error-text"
						>
							O nome da equipe é obrigatório.
						</span>
					</div>

					<!-- Descrição -->
					<div class="form-group">
						<label
							class="form-label"
							for="team-desc"
						>Descrição</label>
						<textarea
							id="team-desc"
							v-model="teamDescription"
							class="form-textarea"
							placeholder="Qual o propósito desta equipe e quais áreas ela atende?"
						/>
					</div>

					<!-- Convidar Membros da Equipe (Autocomplete Dinâmico de Colaboradores) -->
					<div class="form-group autocomplete-group">
						<label class="form-label">Convidar Membros da Equipe</label>
						<div
							class="member-search-box"
							:class="{'is-focused': isSearchFocused}"
							@click="focusSearchInput"
						>
							<!-- Selected Chips -->
							<div
								v-for="member in selectedMembers"
								:key="member.id"
								class="member-chip"
							>
								<div
									class="chip-avatar"
									:style="{backgroundColor: member.color}"
								>
									{{ member.initials }}
								</div>
								<span class="chip-name">{{ member.name }}</span>
								<button
									type="button"
									class="chip-remove"
									aria-label="Remover colaborador"
									@click.stop="removeMember(member.id)"
								>
									&times;
								</button>
							</div>

							<!-- Search Input -->
							<input
								ref="searchInputRef"
								v-model="searchQuery"
								type="text"
								class="chip-input"
								placeholder="Digite o nome ou e-mail..."
								@focus="onSearchFocus"
								@blur="onSearchBlur"
							>
						</div>
						<span class="form-hint">Digite para buscar colaboradores da organização e clique para selecionar.</span>

						<!-- Dropdown de Autocomplete -->
						<div
							v-if="isDropdownVisible"
							class="autocomplete-dropdown"
						>
							<div
								v-if="filteredCollaborators.length === 0"
								class="empty-dropdown"
							>
								Nenhum colaborador encontrado
							</div>
							<div
								v-for="collab in filteredCollaborators"
								:key="collab.id"
								class="collab-item"
								@mousedown.prevent="selectMember(collab)"
							>
								<div
									class="collab-avatar"
									:style="{backgroundColor: collab.color}"
								>
									{{ collab.initials }}
								</div>
								<div class="collab-info">
									<div class="collab-name">
										{{ collab.name }}
									</div>
									<div class="collab-meta">
										{{ collab.email }} • <span class="collab-role">{{ collab.role }}</span>
									</div>
								</div>
							</div>
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
							<span v-else>Criar Equipe</span>
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

export interface Collaborator {
	id: number
	name: string
	email: string
	role: string
	initials: string
	color: string
}

const router = useRouter()

const teamName = ref('')
const teamDescription = ref('')
const searchQuery = ref('')
const isSearchFocused = ref(false)
const searchInputRef = ref<HTMLInputElement | null>(null)
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

const collaboratorsCatalog: Collaborator[] = [
	{id: 1, name: 'Matheus Oliveira', email: 'matheus@empresa.com', role: 'Tech Lead / Eng', initials: 'MO', color: '#5e6ad2'},
	{id: 2, name: 'Danilo Costa', email: 'danilo.costa@empresa.com', role: 'Product Operations', initials: 'DC', color: '#10b981'},
	{id: 3, name: 'Ana Silva', email: 'ana.silva@empresa.com', role: 'UI/UX Designer', initials: 'AS', color: '#ec4899'},
	{id: 4, name: 'Carlos Mendes', email: 'carlos.mendes@empresa.com', role: 'Senior Backend', initials: 'CM', color: '#f59e0b'},
	{id: 5, name: 'Bruna Lima', email: 'bruna.lima@empresa.com', role: 'Frontend Engineer', initials: 'BL', color: '#8b5cf6'},
	{id: 6, name: 'Felipe Rocha', email: 'felipe.rocha@empresa.com', role: 'QA & Compliance', initials: 'FR', color: '#06b6d4'},
]

const selectedMembers = ref<Collaborator[]>([])

const filteredCollaborators = computed(() => {
	const query = searchQuery.value.trim().toLowerCase()
	return collaboratorsCatalog.filter(c => {
		const alreadySelected = selectedMembers.value.some(m => m.id === c.id)
		if (alreadySelected) return false
		if (!query) return true
		return (
			c.name.toLowerCase().includes(query) ||
			c.email.toLowerCase().includes(query) ||
			c.role.toLowerCase().includes(query)
		)
	})
})

const isDropdownVisible = computed(() => {
	return isSearchFocused.value
})

function focusSearchInput() {
	searchInputRef.value?.focus()
}

function onSearchFocus() {
	isSearchFocused.value = true
}

function onSearchBlur() {
	setTimeout(() => {
		isSearchFocused.value = false
	}, 200)
}

function selectMember(collab: Collaborator) {
	selectedMembers.value.push(collab)
	searchQuery.value = ''
	showToast(`${collab.name} adicionado à equipe`)
	focusSearchInput()
}

function removeMember(id: number) {
	selectedMembers.value = selectedMembers.value.filter(m => m.id !== id)
}

function cancel() {
	router.push({name: 'teams.index'})
}

function createTeam() {
	if (!teamName.value.trim()) {
		showError.value = true
		return
	}
	showError.value = false
	isSubmitting.value = true

	showToast(`Equipe "${teamName.value}" criada com sucesso!`)

	setTimeout(() => {
		isSubmitting.value = false
		router.push({name: 'teams.index'})
	}, 600)
}
</script>

<style scoped lang="scss">
.create-team-page {
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
	--accent-chip: rgba(94, 106, 210, 0.10);

	--radius-sm: 6px;
	--radius-md: 8px;
	--radius-lg: 12px;

	--shadow-card: 0 4px 20px rgba(0, 0, 0, 0.05);
	--shadow-dropdown: 0 12px 28px rgba(0, 0, 0, 0.12);

	font-family: var(--font-family);
	background-color: var(--bg-app);
	color: var(--text-primary);
	min-height: 100vh;
	display: flex;
	flex-direction: column;
	width: 100%;
}

:global(html.dark) .create-team-page {
	/* Design Tokens - Dark Theme */
	--bg-app: #0d0e12;
	--bg-sidebar: #12141a;
	--bg-card: #181b22;
	--bg-input: #1f232d;
	--bg-hover: #262a36;

	--text-primary: #f0f2f5;
	--text-secondary: #9aa1b2;
	--text-tertiary: #656c80;

	--border-color: #262a36;
	--border-subtle: #1e222c;
	--border-focus: #5e6ad2;

	--accent-chip: rgba(94, 106, 210, 0.18);

	--shadow-card: 0 4px 20px rgba(0, 0, 0, 0.25);
	--shadow-dropdown: 0 12px 28px rgba(0, 0, 0, 0.45);
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

.autocomplete-group {
	position: relative;
}

.form-label {
	font-size: 12px;
	font-weight: 600;
	color: var(--text-secondary);
	text-transform: uppercase;
	letter-spacing: 0.4px;
}

.form-input,
.form-textarea {
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

/* Member Search Box & Chips */
.member-search-box {
	display: flex;
	flex-wrap: wrap;
	gap: 8px;
	align-items: center;
	background: var(--bg-input);
	border: 1px solid var(--border-color);
	border-radius: var(--radius-md);
	padding: 8px 12px;
	min-height: 46px;
	transition: all 0.15s ease;
	cursor: text;

	&.is-focused {
		border-color: var(--border-focus);
		box-shadow: 0 0 0 2px rgba(94, 106, 210, 0.2);
	}
}

.member-chip {
	display: inline-flex;
	align-items: center;
	gap: 6px;
	background: var(--accent-chip);
	border: 1px solid rgba(94, 106, 210, 0.35);
	padding: 3px 8px 3px 6px;
	border-radius: 6px;
	font-size: 12px;
	color: var(--text-primary);
}

.chip-avatar {
	width: 18px;
	height: 18px;
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 10px;
	font-weight: bold;
	color: #ffffff;
}

.chip-name {
	font-weight: 500;
}

.chip-remove {
	background: transparent;
	border: none;
	cursor: pointer;
	margin-left: 2px;
	color: var(--text-secondary);
	font-weight: bold;
	font-size: 14px;
	line-height: 1;
	padding: 0;

	&:hover {
		color: #ef4444;
	}
}

.chip-input {
	background: transparent;
	border: none;
	outline: none;
	color: var(--text-primary);
	font-family: inherit;
	font-size: 13px;
	flex: 1;
	min-width: 140px;
}

/* Autocomplete Dropdown */
.autocomplete-dropdown {
	position: absolute;
	top: calc(100% - 10px);
	left: 0;
	right: 0;
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: var(--radius-md);
	box-shadow: var(--shadow-dropdown);
	z-index: 50;
	max-height: 220px;
	overflow-y: auto;
	padding: 6px;
}

.empty-dropdown {
	padding: 10px 14px;
	font-size: 13px;
	color: var(--text-tertiary);
	text-align: center;
}

.collab-item {
	display: flex;
	align-items: center;
	gap: 10px;
	padding: 8px 12px;
	border-radius: 6px;
	cursor: pointer;
	transition: background 0.15s ease;

	&:hover {
		background: var(--bg-hover);
	}
}

.collab-avatar {
	width: 30px;
	height: 30px;
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 12px;
	font-weight: 600;
	color: #ffffff;
	flex-shrink: 0;
}

.collab-info {
	flex: 1;
	min-width: 0;
}

.collab-name {
	font-size: 13px;
	font-weight: 600;
	color: var(--text-primary);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.collab-meta {
	font-size: 12px;
	color: var(--text-secondary);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.collab-role {
	color: var(--text-tertiary);
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
