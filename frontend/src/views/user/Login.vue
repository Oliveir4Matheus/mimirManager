<template>
	<div class="login-card-container">
		<div class="login-card">
			<!-- Header -->
			<div class="card-header">
				<div
					class="logo-badge"
					role="img"
					aria-label="Vikunja Logo"
				>
					<svg
						width="18"
						height="18"
						viewBox="0 0 24 24"
						fill="none"
						stroke="#ffffff"
						stroke-width="2.2"
						stroke-linecap="round"
						stroke-linejoin="round"
						aria-hidden="true"
					>
						<path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5" />
					</svg>
				</div>
				<h1 class="card-title">
					{{ $t('user.auth.login') || 'Bem-vindo de volta' }}
				</h1>
				<p class="card-subtitle">
					<span>{{ $t('apiConfig.use', {domain: apiDomain}) || `Instalação ${apiDomain}` }}</span>
					<button
						type="button"
						class="change-api-link"
						@click="toggleApiConfig"
					>
						{{ $t('apiConfig.change') || 'alterar' }}
					</button>
				</p>
			</div>

			<!-- Inline API Configuration Toggle -->
			<div
				v-if="showApiConfig"
				class="api-config-drawer"
			>
				<ApiConfig
					:configure-open="true"
					@found-api="onApiFound"
				/>
			</div>

			<!-- Feedback Messages -->
			<div
				v-if="confirmedEmailSuccess"
				class="alert alert-success"
				role="status"
			>
				{{ $t('user.auth.confirmEmailSuccess') }}
			</div>
			<div
				v-if="errorMessage"
				class="alert alert-danger"
				role="alert"
			>
				{{ errorMessage }}
			</div>

			<DesktopLogin v-if="isDesktop" />

			<form
				v-if="!isDesktop && (localAuthEnabled || ldapAuthEnabled)"
				id="loginform"
				novalidate
				@submit.prevent="submit"
			>
				<!-- Username/Email Field -->
				<div class="form-group">
					<label
						for="username"
						class="form-label"
					>
						{{ $t('user.auth.usernameEmail') || 'Nome de usuário ou E-mail' }}
					</label>
					<input
						id="username"
						ref="usernameRef"
						v-model="username"
						v-focus
						type="text"
						name="username"
						autocomplete="username"
						placeholder="ex: frederick"
						class="form-input"
						:class="{'is-invalid': hasAttemptedSubmit && !usernameValid}"
						:aria-invalid="hasAttemptedSubmit && !usernameValid ? 'true' : undefined"
						:aria-describedby="hasAttemptedSubmit && !usernameValid ? 'username-error' : undefined"
						@input="validateUsernameField"
						@focusout="validateUsernameField"
					>
					<span
						v-if="hasAttemptedSubmit && !usernameValid"
						id="username-error"
						class="field-error"
						role="alert"
					>
						{{ $t('user.auth.usernameRequired') || 'O nome de usuário ou e-mail é obrigatório' }}
					</span>
				</div>

				<!-- Password Field -->
				<div class="form-group">
					<div class="label-row">
						<label
							for="password"
							class="form-label"
						>
							{{ $t('user.auth.password') || 'Senha' }}
						</label>
						<RouterLink
							v-if="localAuthEnabled"
							:to="{ name: 'user.password-reset.request' }"
							class="forgot-password-link"
						>
							{{ $t('user.auth.forgotPassword') || 'Esqueceu a senha?' }}
						</RouterLink>
					</div>

					<div class="password-input-wrapper">
						<input
							id="password"
							ref="passwordRef"
							v-model="password"
							:type="passwordVisible ? 'text' : 'password'"
							name="password"
							autocomplete="current-password"
							placeholder="Sua senha"
							class="form-input password-input"
							:class="{'is-invalid': hasAttemptedSubmit && !passwordValid}"
							:aria-invalid="hasAttemptedSubmit && !passwordValid ? 'true' : undefined"
							:aria-describedby="hasAttemptedSubmit && !passwordValid ? 'password-error' : undefined"
							@input="validatePasswordField"
							@focusout="validatePasswordField"
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
						v-if="hasAttemptedSubmit && !passwordValid"
						id="password-error"
						class="field-error"
						role="alert"
					>
						{{ $t('user.auth.passwordRequired') || 'A senha é obrigatória' }}
					</span>
				</div>

				<!-- TOTP Passcode Field -->
				<div
					v-if="needsTotpPasscode"
					class="form-group"
				>
					<label
						for="totpPasscode"
						class="form-label"
					>
						{{ $t('user.auth.totpTitle') || 'Código de autenticação em duas etapas' }}
					</label>
					<input
						id="totpPasscode"
						ref="totpPasscodeRef"
						v-focus
						v-model="totpPasscodeValue"
						type="text"
						inputmode="numeric"
						autocomplete="one-time-code"
						:placeholder="$t('user.auth.totpPlaceholder') || 'Código TOTP'"
						class="form-input"
						required
					>
				</div>

				<!-- Remember Me Checkbox -->
				<div class="checkbox-row">
					<input
						id="rememberMe"
						v-model="rememberMe"
						type="checkbox"
						class="form-checkbox"
					>
					<label
						for="rememberMe"
						class="checkbox-label"
					>
						{{ $t('user.auth.remember') || 'Permanecer conectado' }}
					</label>
				</div>

				<!-- Primary Login Button -->
				<button
					type="submit"
					class="btn-primary"
					:disabled="isLoading"
				>
					<span
						v-if="isLoading"
						class="btn-spinner"
						aria-hidden="true"
					/>
					<span v-else>{{ $t('user.auth.login') || 'Entrar' }}</span>
				</button>

				<!-- Footer -->
				<div
					v-if="registrationEnabled"
					class="card-footer"
				>
					<span>{{ $t('user.auth.noAccountYet') || 'Não tem uma conta?' }}</span>
					<RouterLink
						:to="{ name: 'user.register' }"
						class="create-account-link"
					>
						{{ $t('user.auth.createAccount') || 'Criar conta' }}
					</RouterLink>
				</div>
			</form>

			<!-- OpenID Connect Providers -->
			<div
				v-if="!isDesktop && hasOpenIdProviders"
				class="openid-section"
			>
				<div class="divider">
					<span>{{ $t('user.auth.orLoginWith') || 'ou continue com' }}</span>
				</div>
				<button
					v-for="(p, k) in openidConnect.providers"
					:key="k"
					type="button"
					class="btn-openid"
					@click="redirectToProvider(p)"
				>
					{{ $t('user.auth.loginWith', {provider: p.name}) }}
				</button>
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
import type {VikunjaErrorModel} from '@/client/generated'
import {computed, onBeforeMount, ref} from 'vue'
import {useI18n} from 'vue-i18n'
import {useRoute, useRouter} from 'vue-router'
import {parseURL} from 'ufo'

import ApiConfig from '@/components/misc/ApiConfig.vue'
import DesktopLogin from '@/views/user/DesktopLogin.vue'
import Icon from '@/components/misc/Icon'

import {getErrorText} from '@/message'
import {getAutoRedirectProvider, redirectToProvider} from '@/helpers/redirectToProvider'
import {useRedirectToLastVisited} from '@/composables/useRedirectToLastVisited'
import {isDesktopApp} from '@/helpers/desktopAuth'
import {REDIRECT_HASH_PREFIX} from '@/constants/redirectHash'
import {ERROR_CODE_TOTP_REQUIRED} from '@/constants/auth'

import {useAuthStore, JUST_LOGGED_OUT_KEY} from '@/stores/auth'
import {useConfigStore} from '@/stores/config'
import {useTitle} from '@/composables/useTitle'

const {t} = useI18n({useScope: 'global'})
useTitle(() => t('user.auth.login') || 'Login')

const route = useRoute()
const router = useRouter()
const authStore = useAuthStore()
const configStore = useConfigStore()
const {redirectIfSaved} = useRedirectToLastVisited()

const registrationEnabled = computed(() => configStore.auth.local.registration_enabled)
const localAuthEnabled = computed(() => configStore.auth.local.enabled)
const ldapAuthEnabled = computed(() => configStore.auth.ldap.enabled)

const openidConnect = computed(() => configStore.auth.openid_connect)
const hasOpenIdProviders = computed(() => openidConnect.value.enabled && (openidConnect.value.providers?.length ?? 0) > 0)

const isLoading = computed(() => authStore.isLoading)
const isDesktop = isDesktopApp()

const confirmedEmailSuccess = ref(false)
const errorMessage = ref('')
const username = ref('')
const password = ref('')
const totpPasscodeValue = ref('')
const rememberMe = ref(false)
const passwordVisible = ref(false)
const showApiConfig = ref(false)
const hasAttemptedSubmit = ref(false)

const usernameRef = ref<HTMLInputElement | null>(null)
const passwordRef = ref<HTMLInputElement | null>(null)
const totpPasscodeRef = ref<HTMLInputElement | null>(null)

const apiUrl = ref(window.API_URL)
const apiDomain = computed(() => parseURL(apiUrl.value, 'http://').host || parseURL(window.location.href).host)

function toggleApiConfig() {
	showApiConfig.value = !showApiConfig.value
}

function onApiFound(url: string) {
	apiUrl.value = url
	showApiConfig.value = false
}

const usernameValid = computed(() => username.value.trim() !== '')
const passwordValid = computed(() => password.value !== '')

function validateUsernameField() {
	// reactive through computed
}

function validatePasswordField() {
	// reactive through computed
}

const authenticated = computed(() => authStore.authenticated)
const needsTotpPasscode = computed(() => authStore.needsTotpPasscode)

onBeforeMount(() => {
	authStore.verifyEmail().then((confirmed) => {
		confirmedEmailSuccess.value = confirmed
	}).catch((e: Error) => {
		errorMessage.value = e.message
	})

	if (authenticated.value) {
		router.push({name: 'home'})
		return
	}

	const justLoggedOut = sessionStorage.getItem(JUST_LOGGED_OUT_KEY) !== null
	if (justLoggedOut) {
		sessionStorage.removeItem(JUST_LOGGED_OUT_KEY)
	}

	const autoRedirectProvider = getAutoRedirectProvider({
		localAuthEnabled: localAuthEnabled.value,
		ldapAuthEnabled: ldapAuthEnabled.value,
		openIdEnabled: openidConnect.value.enabled,
		providers: openidConnect.value.providers ?? [],
		isDesktopApp: isDesktop,
		justLoggedOut,
		hasCopyableRedirect: route.hash.startsWith(REDIRECT_HASH_PREFIX),
	})
	if (autoRedirectProvider) {
		redirectToProvider(autoRedirectProvider)
	}
})

async function submit() {
	errorMessage.value = ''
	hasAttemptedSubmit.value = true

	// Also support autofilled values in case v-model did not catch it
	const inputUsername = username.value || usernameRef.value?.value || ''
	const inputPassword = password.value || passwordRef.value?.value || ''

	if (inputUsername.trim() === '' || inputPassword === '') {
		return
	}

	const credentials: Parameters<typeof authStore.login>[0] = {
		username: inputUsername,
		password: inputPassword,
		longToken: rememberMe.value,
	}

	if (needsTotpPasscode.value) {
		credentials.totpPasscode = totpPasscodeValue.value || totpPasscodeRef.value?.value
	}

	try {
		await authStore.login(credentials)
		authStore.setNeedsTotpPasscode(false)
		redirectIfSaved()
	} catch (e) {
		if ((e as VikunjaErrorModel)?.code === ERROR_CODE_TOTP_REQUIRED && !credentials.totpPasscode) {
			return
		}
		errorMessage.value = getErrorText(e)
	}
}
</script>

<style lang="scss" scoped>
.login-card-container {
	/* Design Tokens */
	--bg-page: #f9f9f9;
	--bg-card: #ffffff;
	--text-primary: #111111;
	--text-muted: #6b6b6b;
	--border-default: #e5e5e5;
	--input-bg: #ffffff;
	--accent-base: #5e6ad2;
	--accent-hover: #4a55b5;
	--accent-ring: rgba(94, 106, 210, 0.2);
	--font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;

	font-family: var(--font-family);
	display: flex;
	align-items: center;
	justify-content: center;
	min-height: 100vh;
	width: 100%;
	background-color: var(--bg-page);
	box-sizing: border-box;
}

:global(html.dark) .login-card-container {
	--bg-page: #0f0f0f;
	--bg-card: #1c1c1c;
	--text-primary: #e5e5e5;
	--text-muted: #8a8a8a;
	--border-default: #333333;
	--input-bg: #0f0f0f;
}

@media (prefers-color-scheme: dark) {
	:global(html:not(.light)) .login-card-container {
		--bg-page: #0f0f0f;
		--bg-card: #1c1c1c;
		--text-primary: #e5e5e5;
		--text-muted: #8a8a8a;
		--border-default: #333333;
		--input-bg: #0f0f0f;
	}
}

.login-card {
	width: 340px;
	max-width: 100%;
	padding: 32px;
	border-radius: 8px;
	border: 1px solid var(--border-default);
	background-color: var(--bg-card);
	box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05), 0 1px 2px rgba(0, 0, 0, 0.1);
	box-sizing: border-box;

	@media (max-width: 480px) {
		width: 100%;
		border-radius: 0;
		box-shadow: none;
		border: none;
		background-color: var(--bg-page);
		padding: 24px;
	}
}

.card-header {
	text-align: center;
	margin-bottom: 24px;
}

.logo-badge {
	width: 32px;
	height: 32px;
	border-radius: 6px;
	background: var(--accent-base);
	padding: 4px;
	margin: 0 auto 12px;
	display: flex;
	align-items: center;
	justify-content: center;
	box-sizing: border-box;
}

.card-title {
	font-size: 18px;
	font-weight: 600;
	letter-spacing: -0.3px;
	margin: 0 0 4px 0;
	color: var(--text-primary);
}

.card-subtitle {
	font-size: 13px;
	font-weight: 400;
	margin: 0;
	color: var(--text-muted);
	display: flex;
	align-items: center;
	justify-content: center;
	gap: 4px;
}

.change-api-link {
	background: none;
	border: none;
	padding: 0;
	font-size: 13px;
	font-weight: 400;
	color: var(--accent-base);
	cursor: pointer;
	text-decoration: none;

	&:hover {
		text-decoration: underline;
	}
}

.api-config-drawer {
	margin-bottom: 16px;
	padding: 12px;
	border-radius: 6px;
	background-color: var(--input-bg);
	border: 1px solid var(--border-default);
}

.alert {
	padding: 10px 12px;
	font-size: 13px;
	border-radius: 6px;
	margin-bottom: 16px;
	line-height: 1.4;

	&.alert-danger {
		background-color: rgba(239, 68, 68, 0.1);
		border: 1px solid rgba(239, 68, 68, 0.3);
		color: #ef4444;
	}

	&.alert-success {
		background-color: rgba(34, 197, 94, 0.1);
		border: 1px solid rgba(34, 197, 94, 0.3);
		color: #22c55e;
	}
}

.form-group {
	margin-bottom: 16px;
}

.form-label {
	display: block;
	font-size: 13px;
	font-weight: 500;
	color: var(--text-primary);
	margin-bottom: 6px;
}

.label-row {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 6px;

	.form-label {
		margin-bottom: 0;
	}
}

.forgot-password-link {
	font-size: 13px;
	font-weight: 400;
	color: var(--accent-base);
	text-decoration: none;

	&:hover {
		text-decoration: underline;
	}
}

.form-input {
	width: 100%;
	padding: 8px 12px;
	font-size: 14px;
	font-weight: 400;
	font-family: inherit;
	border-radius: 6px;
	border: 1px solid var(--border-default);
	background-color: var(--input-bg);
	color: var(--text-primary);
	box-sizing: border-box;
	outline: none;
	transition: border-color 0.15s, box-shadow 0.15s;

	&::placeholder {
		color: var(--text-muted);
		opacity: 0.8;
	}

	&:focus {
		border-color: var(--accent-base);
		box-shadow: 0 0 0 3px var(--accent-ring);
	}

	&.is-invalid {
		border-color: #ef4444;

		&:focus {
			box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.2);
		}
	}
}

.password-input-wrapper {
	position: relative;
	display: flex;
	align-items: center;

	.password-input {
		padding-right: 36px;
	}
}

.password-toggle-btn {
	position: absolute;
	right: 10px;
	background: none;
	border: none;
	padding: 0;
	color: var(--text-muted);
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

.checkbox-row {
	display: flex;
	align-items: center;
	gap: 8px;
	margin-bottom: 24px;
}

.form-checkbox {
	width: 16px;
	height: 16px;
	accent-color: var(--accent-base);
	cursor: pointer;
	margin: 0;
}

.checkbox-label {
	font-size: 13px;
	font-weight: 400;
	color: var(--text-primary);
	cursor: pointer;
	user-select: none;
}

.btn-primary {
	width: 100%;
	padding: 9px 16px;
	font-size: 14px;
	font-weight: 500;
	color: #ffffff;
	background-color: var(--accent-base);
	border: none;
	border-radius: 6px;
	cursor: pointer;
	transition: background-color 0.15s;
	font-family: inherit;
	display: flex;
	align-items: center;
	justify-content: center;

	&:hover:not(:disabled) {
		background-color: var(--accent-hover);
	}

	&:disabled {
		opacity: 0.7;
		cursor: not-allowed;
	}
}

.btn-spinner {
	width: 16px;
	height: 16px;
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
	font-size: 13px;
	margin-top: 24px;
	color: var(--text-primary);

	.create-account-link {
		color: var(--accent-base);
		text-decoration: none;
		margin-left: 4px;

		&:hover {
			text-decoration: underline;
		}
	}
}

.openid-section {
	margin-top: 20px;

	.divider {
		position: relative;
		text-align: center;
		margin: 16px 0;

		&::before {
			content: "";
			position: absolute;
			left: 0;
			top: 50%;
			width: 100%;
			height: 1px;
			background-color: var(--border-default);
		}

		span {
			position: relative;
			background-color: var(--bg-card);
			padding: 0 8px;
			font-size: 12px;
			color: var(--text-muted);
		}
	}

	.btn-openid {
		width: 100%;
		padding: 8px 16px;
		font-size: 13px;
		font-weight: 500;
		color: var(--text-primary);
		background-color: var(--input-bg);
		border: 1px solid var(--border-default);
		border-radius: 6px;
		cursor: pointer;
		margin-bottom: 8px;
		transition: background-color 0.15s, border-color 0.15s;

		&:hover {
			border-color: var(--accent-base);
		}
	}
}
</style>
