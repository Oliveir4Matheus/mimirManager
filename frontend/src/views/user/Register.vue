<template>
	<div class="register-page-wrapper">
		<div class="register-card">
			<!-- Header -->
			<div class="card-header">
				<div
					class="logo-badge"
					role="img"
					aria-label="Vikunja Logo"
				>
					V
				</div>
				<h1 class="card-title">
					{{ $t('user.auth.createAccount') || 'Crie sua conta no Vikunja' }}
				</h1>
				<p class="card-subtitle">
					Organize projetos e tarefas com alta produtividade e clareza visual.
				</p>
			</div>

			<!-- Invite state: loading or invalid -->
			<template v-if="isInvite && inviteState.status !== 'ready'">
				<div
					v-if="inviteState.status === 'loading'"
					class="alert alert-info"
				>
					{{ $t('misc.loading') }}
				</div>
				<template v-else>
					<div
						class="alert alert-danger"
						role="alert"
					>
						{{ $t(inviteState.status === 'invalid' ? 'user.auth.inviteInvalid' : 'user.auth.inviteLoadFailed') }}
					</div>
					<div class="card-footer">
						<RouterLink
							:to="{name: 'user.login'}"
							class="login-link"
						>
							{{ $t('user.auth.login') }}
						</RouterLink>
					</div>
				</template>
			</template>

			<!-- Registration form -->
			<div v-else-if="isInvite || configStore.auth.local.registration_enabled">
				<div
					v-if="isInvite && inviteState.status === 'ready' && inviteState.link.teams?.length"
					class="alert alert-info"
				>
					{{ $t('user.auth.inviteTeams', {teams: inviteState.link.teams.map(team => team.name).join(', ')}) }}
				</div>
				<div
					v-if="errorMessage !== ''"
					class="alert alert-danger"
					role="alert"
				>
					{{ errorMessage }}
				</div>
				<div
					v-if="confirmEmailMessage !== ''"
					class="alert alert-success"
					role="status"
				>
					{{ confirmEmailMessage }}
				</div>

				<form
					id="register-form"
					novalidate
					@submit.prevent="submit"
				>
					<!-- Username Field -->
					<div class="form-group">
						<label
							class="form-label"
							for="reg-username"
						>
							{{ $t('user.auth.username') || 'Nome de Usuário' }}
						</label>
						<input
							id="reg-username"
							ref="usernameRef"
							v-model="credentials.username"
							v-focus
							type="text"
							name="username"
							autocomplete="username"
							placeholder="seunome"
							class="form-input"
							:class="{'is-invalid': hasAttemptedSubmit && usernameError}"
							:aria-invalid="hasAttemptedSubmit && usernameError ? 'true' : undefined"
							:aria-describedby="hasAttemptedSubmit && usernameError ? 'reg-username-error' : undefined"
							required
							@input="handleUsernameInput"
							@focusout="validateUsername(); validateUsernameAfterFirst = true"
							@keyup="handleUsernameKeyup"
						>
						<span
							v-if="hasAttemptedSubmit && usernameError"
							id="reg-username-error"
							class="field-error"
							role="alert"
						>
							{{ usernameError }}
						</span>
					</div>

					<!-- Email Field -->
					<div class="form-group">
						<label
							class="form-label"
							for="reg-email"
						>
							{{ $t('user.auth.email') || 'E-mail Corporativo ou Pessoal' }}
						</label>
						<input
							id="reg-email"
							ref="emailRef"
							v-model="credentials.email"
							type="email"
							name="email"
							autocomplete="email"
							placeholder="nome@empresa.com"
							class="form-input"
							:class="{'is-invalid': hasAttemptedSubmit && emailError}"
							:aria-invalid="hasAttemptedSubmit && emailError ? 'true' : undefined"
							:aria-describedby="hasAttemptedSubmit && emailError ? 'reg-email-error' : undefined"
							required
							@input="handleEmailInput"
							@focusout="validateEmail(); validateEmailAfterFirst = true"
							@keyup="handleEmailKeyup"
						>
						<span
							v-if="hasAttemptedSubmit && emailError"
							id="reg-email-error"
							class="field-error"
							role="alert"
						>
							{{ emailError }}
						</span>
					</div>

					<!-- Password Field -->
					<div class="form-group">
						<label
							class="form-label"
							for="reg-password"
						>
							{{ $t('user.auth.password') || 'Senha' }}
						</label>
						<div class="password-input-wrapper">
							<input
								id="reg-password"
								ref="passwordRef"
								v-model="credentials.password"
								:type="passwordVisible ? 'text' : 'password'"
								name="password"
								autocomplete="new-password"
								placeholder="Mínimo 8 caracteres"
								class="form-input password-input"
								:class="{'is-invalid': hasAttemptedSubmit && passwordError}"
								:aria-invalid="hasAttemptedSubmit && passwordError ? 'true' : undefined"
								:aria-describedby="hasAttemptedSubmit && passwordError ? 'reg-password-error' : undefined"
								required
								@input="handlePasswordInput"
								@focusout="validatePasswordInitially = true"
							>
							<button
								type="button"
								class="password-toggle-btn"
								:aria-label="passwordVisible ? ($t('user.auth.hidePassword') || 'Ocultar senha') : ($t('user.auth.showPassword') || 'Mostrar senha')"
								@click="passwordVisible = !passwordVisible"
							>
								<Icon :icon="passwordVisible ? 'eye-slash' : 'eye'" />
							</button>
						</div>
						<span
							v-if="hasAttemptedSubmit && passwordError"
							id="reg-password-error"
							class="field-error"
							role="alert"
						>
							{{ passwordError }}
						</span>
					</div>

					<!-- Submit Button -->
					<button
						id="register-submit"
						type="submit"
						class="btn-primary"
						:disabled="isLoading"
					>
						<span
							v-if="isLoading"
							class="btn-spinner"
							aria-hidden="true"
						/>
						<span v-else>{{ $t('user.auth.createAccount') || 'Criar Conta' }}</span>
					</button>

					<!-- Demo Mode Notice -->
					<div
						v-if="configStore.demo_mode_enabled"
						class="alert alert-warning mbs-4"
					>
						<strong>{{ $t('demo.title') }}:</strong>
						{{ $t('demo.accountWillBeDeleted') }}<br>
						<span class="is-uppercase">{{ $t('demo.everythingWillBeDeleted') }}</span>
					</div>
				</form>

				<!-- Footer -->
				<div class="card-footer">
					<span>{{ $t('user.auth.alreadyHaveAnAccount') || 'Já tem uma conta?' }}</span>
					<RouterLink
						:to="{ name: 'user.login' }"
						class="login-link"
					>
						{{ $t('user.auth.login') || 'Fazer login' }}
					</RouterLink>
				</div>
			</div>

			<!-- Registration Disabled Message -->
			<div
				v-else
				class="alert alert-warning"
			>
				{{ $t('user.auth.registrationDisabled') }}
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
import {useDebounceFn} from '@vueuse/core'
import {computed, onBeforeMount, onUnmounted, reactive, ref, toRaw} from 'vue'
import {useI18n} from 'vue-i18n'

import router from '@/router'
import type {PublicInviteLink} from '@/client/generated'
import {hasInviteLink, checkInviteLink, onInviteLinkChange} from '@/client/inviteLink'
import Icon from '@/components/misc/Icon'
import {isEmail} from '@/helpers/isEmail'
import {parseValidationErrors, type ValidationError} from '@/helpers/parseValidationErrors'

import {useRedirectToLastVisited} from '@/composables/useRedirectToLastVisited'
import {useAuthStore} from '@/stores/auth'
import {useConfigStore} from '@/stores/config'
import {validatePassword} from '@/helpers/validatePasswort'

const isInvite = ref(hasInviteLink())
type InviteState = {status: 'loading' | 'invalid' | 'error'} | {status: 'ready', link: PublicInviteLink}
const inviteState = ref<InviteState>({status: 'loading'})
let inviteCheck = 0

async function loadInvite() {
	isInvite.value = hasInviteLink()
	if (!isInvite.value) return
	const check = ++inviteCheck
	inviteState.value = {status: 'loading'}
	try {
		const {data: link} = await checkInviteLink()
		if (check !== inviteCheck) return
		inviteState.value = {status: 'ready', link}
	} catch (e) {
		if (check !== inviteCheck) return
		inviteState.value = {status: e instanceof Object && 'status' in e && e.status === 404 ? 'invalid' : 'error'}
	}
}

onBeforeMount(loadInvite)
const unsubscribeInvite = onInviteLinkChange(loadInvite)
onUnmounted(() => {
	inviteCheck++
	unsubscribeInvite()
})

const {t} = useI18n()
const authStore = useAuthStore()
const configStore = useConfigStore()
const {redirectIfSaved} = useRedirectToLastVisited()

onBeforeMount(() => {
	if (authStore.authenticated) {
		router.push({name: 'home'})
	}
})

const credentials = reactive({
	username: '',
	email: '',
	password: '',
})

const isLoading = computed(() => authStore.isLoading)
const errorMessage = ref('')
const confirmEmailMessage = ref('')
const validatePasswordInitially = ref(false)
const serverValidationErrors = ref<Partial<Record<string, string>>>({})
const passwordVisible = ref(false)
const hasAttemptedSubmit = ref(false)

const usernameRef = ref<HTMLInputElement | null>(null)
const emailRef = ref<HTMLInputElement | null>(null)
const passwordRef = ref<HTMLInputElement | null>(null)

const DEBOUNCE_TIME = 100

const emailValid = ref(true)
const validateEmailAfterFirst = ref(false)
const validateEmail = useDebounceFn(() => {
	emailValid.value = isEmail(credentials.email)
}, DEBOUNCE_TIME)

const usernameValid = ref<true | string>(true)
const validateUsernameAfterFirst = ref(false)
const validateUsername = useDebounceFn(() => {
	if (credentials.username === '') {
		usernameValid.value = t('user.auth.usernameRequired') || 'O nome de usuário é obrigatório'
		return
	}

	if (credentials.username.indexOf(' ') !== -1) {
		usernameValid.value = t('user.auth.usernameMustNotContainSpace') || 'O nome de usuário não pode conter espaços'
		return
	}

	if (credentials.username.indexOf('://') !== -1 || credentials.username.indexOf('.') !== -1) {
		usernameValid.value = t('user.auth.usernameMustNotLookLikeUrl') || 'O nome de usuário não pode conter URLs'
		return
	}

	usernameValid.value = true
}, DEBOUNCE_TIME)

const everythingValid = computed(() => {
	return credentials.username.trim() !== '' &&
		credentials.email.trim() !== '' &&
		validatePassword(credentials.password) === true &&
		emailValid.value &&
		usernameValid.value === true
})

const usernameError = computed(() => {
	if (usernameValid.value !== true) {
		return usernameValid.value
	}
	return serverValidationErrors.value.username || null
})

const emailError = computed(() => {
	if (!emailValid.value) {
		return t('user.auth.emailInvalid') || 'E-mail inválido'
	}
	return serverValidationErrors.value.email || null
})

const passwordError = computed(() => {
	if (validatePasswordInitially.value) {
		const res = validatePassword(credentials.password)
		if (res !== true) {
			return t(res) || 'A senha deve ter no mínimo 8 caracteres'
		}
	}
	return serverValidationErrors.value.password || null
})

function handleUsernameInput() {
	delete serverValidationErrors.value.username
	if (validateUsernameAfterFirst.value) {
		validateUsername()
	}
}

function handleUsernameKeyup() {
	if (validateUsernameAfterFirst.value) {
		validateUsername()
	}
	delete serverValidationErrors.value.username
}

function handleEmailInput() {
	delete serverValidationErrors.value.email
	if (validateEmailAfterFirst.value) {
		validateEmail()
	}
}

function handleEmailKeyup() {
	if (validateEmailAfterFirst.value) {
		validateEmail()
	}
	delete serverValidationErrors.value.email
}

function handlePasswordInput() {
	delete serverValidationErrors.value.password
}

function isApiValidationError(error: unknown): error is ValidationError {
	return error !== null &&
		typeof error === 'object' &&
		('invalid_fields' in error || 'errors' in error)
}

async function submit() {
	errorMessage.value = ''
	serverValidationErrors.value = {}
	validatePasswordInitially.value = true
	hasAttemptedSubmit.value = true

	validateUsername()
	validateEmail()

	if (!everythingValid.value || (isInvite.value && inviteState.value.status !== 'ready')) {
		return
	}

	try {
		if (isInvite.value) {
			await authStore.registerWithInvite(toRaw(credentials))
		} else {
			await authStore.register(toRaw(credentials))
		}
		redirectIfSaved()
	} catch (e: unknown) {
		if (isInvite.value && e instanceof Object && 'code' in e && e.code === 2005) {
			inviteState.value = {status: 'invalid'}
			return
		}
		if (e instanceof Object && 'code' in e && e.code === 1012) {
			confirmEmailMessage.value = t('user.auth.registrationConfirmEmail') || 'E-mail de confirmação enviado'
			return
		}

		if (isApiValidationError(e)) {
			const fieldErrors = parseValidationErrors(e)
			if (Object.keys(fieldErrors).length > 0) {
				serverValidationErrors.value = fieldErrors
			} else {
				errorMessage.value = t('user.auth.registrationFailed') || 'Falha no cadastro'
			}
		} else if (e instanceof Object && 'message' in e && typeof e.message === 'string') {
			errorMessage.value = e.message
		} else {
			errorMessage.value = t('user.auth.registrationFailed') || 'Falha no cadastro'
		}
	}
}
</script>

<style lang="scss" scoped>
.register-page-wrapper {
	/* Design Tokens - Light Theme Default */
	--font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
	--bg-app: #f9f9f9;
	--bg-card: #ffffff;
	--bg-input: #ffffff;
	--bg-hover: #f4f4f5;
	--text-primary: #111111;
	--text-secondary: #6b6b6b;
	--text-tertiary: #9ca3af;
	--border-color: #e5e5e5;
	--border-subtle: #f0f0f0;
	--border-focus: #5e6ad2;
	--accent-base: #5e6ad2;
	--accent-hover: #4b56b8;
	--accent-gradient: linear-gradient(135deg, #5e6ad2 0%, #8b5cf6 100%);
	--shadow-card: 0 8px 24px rgba(0, 0, 0, 0.06);

	font-family: var(--font-family);
	display: flex;
	align-items: center;
	justify-content: center;
	min-height: 100vh;
	width: 100%;
	padding: 24px;
	box-sizing: border-box;
	background: radial-gradient(circle at 50% 20%, rgba(94, 106, 210, 0.06), transparent 70%), var(--bg-app);
}

:global(html.dark) .register-page-wrapper {
	--bg-app: #0d0e12;
	--bg-card: #181b22;
	--bg-input: #1f232d;
	--bg-hover: #262a36;
	--text-primary: #f0f2f5;
	--text-secondary: #9aa1b2;
	--text-tertiary: #656c80;
	--border-color: #232836;
	--border-subtle: #1c202c;
	--border-focus: #5e6ad2;
	--shadow-card: 0 12px 32px rgba(0, 0, 0, 0.4);
	background: radial-gradient(circle at 50% 20%, rgba(94, 106, 210, 0.15), transparent 70%), var(--bg-app);
}

@media (prefers-color-scheme: dark) {
	:global(html:not(.light)) .register-page-wrapper {
		--bg-app: #0d0e12;
		--bg-card: #181b22;
		--bg-input: #1f232d;
		--bg-hover: #262a36;
		--text-primary: #f0f2f5;
		--text-secondary: #9aa1b2;
		--text-tertiary: #656c80;
		--border-color: #232836;
		--border-subtle: #1c202c;
		--border-focus: #5e6ad2;
		--shadow-card: 0 12px 32px rgba(0, 0, 0, 0.4);
		background: radial-gradient(circle at 50% 20%, rgba(94, 106, 210, 0.15), transparent 70%), var(--bg-app);
	}
}

.register-card {
	width: 100%;
	max-width: 440px;
	background: var(--bg-card);
	border: 1px solid var(--border-color);
	border-radius: 12px;
	padding: 40px;
	box-shadow: var(--shadow-card);
	box-sizing: border-box;

	@media (max-width: 480px) {
		padding: 24px;
		border-radius: 8px;
	}
}

.card-header {
	display: flex;
	flex-direction: column;
	align-items: center;
	text-align: center;
	margin-bottom: 32px;
}

.logo-badge {
	width: 48px;
	height: 48px;
	background: var(--accent-gradient);
	border-radius: 10px;
	display: flex;
	align-items: center;
	justify-content: center;
	font-weight: 700;
	font-size: 22px;
	color: #ffffff;
	margin-bottom: 16px;
	box-shadow: 0 4px 16px rgba(94, 106, 210, 0.4);
	user-select: none;
}

.card-title {
	font-size: 22px;
	font-weight: 700;
	letter-spacing: -0.5px;
	color: var(--text-primary);
	margin: 0 0 8px 0;
}

.card-subtitle {
	font-size: 13px;
	font-weight: 400;
	color: var(--text-secondary);
	line-height: 1.5;
	margin: 0;
}

.alert {
	padding: 10px 14px;
	font-size: 13px;
	border-radius: 6px;
	margin-bottom: 20px;
	line-height: 1.4;

	&.alert-danger {
		background-color: rgba(239, 68, 68, 0.12);
		border: 1px solid rgba(239, 68, 68, 0.3);
		color: #ef4444;
	}

	&.alert-success {
		background-color: rgba(34, 197, 94, 0.12);
		border: 1px solid rgba(34, 197, 94, 0.3);
		color: #22c55e;
	}

	&.alert-info {
		background-color: rgba(94, 106, 210, 0.12);
		border: 1px solid rgba(94, 106, 210, 0.3);
		color: var(--accent-base);
	}

	&.alert-warning {
		background-color: rgba(234, 179, 8, 0.12);
		border: 1px solid rgba(234, 179, 8, 0.3);
		color: #eab308;
	}
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

.form-input {
	height: 42px;
	background: var(--bg-input);
	border: 1px solid var(--border-color);
	border-radius: 6px;
	padding: 10px 14px;
	color: var(--text-primary);
	font-family: inherit;
	font-size: 14px;
	outline: none;
	box-sizing: border-box;
	transition: border-color 0.15s ease, box-shadow 0.15s ease;

	&::placeholder {
		color: var(--text-tertiary);
	}

	&:focus {
		border-color: var(--border-focus);
		box-shadow: 0 0 0 2px rgba(94, 106, 210, 0.2);
	}

	&.is-invalid {
		border-color: #ef4444;

		&:focus {
			box-shadow: 0 0 0 2px rgba(239, 68, 68, 0.2);
		}
	}
}

.password-input-wrapper {
	position: relative;
	display: flex;
	align-items: center;

	.password-input {
		width: 100%;
		padding-right: 40px;
	}
}

.password-toggle-btn {
	position: absolute;
	right: 12px;
	background: none;
	border: none;
	padding: 0;
	color: var(--text-tertiary);
	cursor: pointer;
	display: flex;
	align-items: center;
	justify-content: center;

	&:hover {
		color: var(--text-primary);
	}
}

.field-error {
	font-size: 12px;
	color: #ef4444;
	margin-top: 4px;
	display: block;
}

.btn-primary {
	width: 100%;
	height: 44px;
	margin-top: 12px;
	background: var(--accent-base);
	color: #ffffff;
	font-weight: 600;
	font-size: 14px;
	border: none;
	border-radius: 6px;
	cursor: pointer;
	font-family: inherit;
	display: flex;
	align-items: center;
	justify-content: center;
	transition: background-color 0.15s ease, transform 0.15s ease, box-shadow 0.15s ease;

	&:hover:not(:disabled) {
		background: var(--accent-hover);
		transform: translateY(-1px);
		box-shadow: 0 4px 14px rgba(94, 106, 210, 0.4);
	}

	&:disabled {
		opacity: 0.7;
		cursor: not-allowed;
	}
}

.btn-spinner {
	width: 18px;
	height: 18px;
	border: 2px solid rgba(255, 255, 255, 0.3);
	border-top-color: #ffffff;
	border-radius: 50%;
	animation: spin 0.8s linear infinite;
}

@keyframes spin {
	to {
		transform: rotate(360deg);
	}
}

.card-footer {
	text-align: center;
	margin-top: 24px;
	padding-top: 20px;
	border-top: 1px solid var(--border-subtle);
	font-size: 13px;
	color: var(--text-secondary);

	.login-link {
		color: var(--accent-base);
		text-decoration: none;
		font-weight: 600;
		margin-left: 4px;

		&:hover {
			text-decoration: underline;
		}
	}
}
</style>
