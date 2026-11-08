<script setup lang="ts">
import {useQuery} from '@tanstack/vue-query'
import {apiTokensQuery, useDeleteApiTokenMutation} from '@/client/queries/apiTokens'
import {isApiTokenExpired} from '@/helpers/apiToken'
import {computed, onMounted, ref} from 'vue'
import {useRoute} from 'vue-router'
import {useRouteQuery} from '@vueuse/router'
import {normalizePageNumber} from '@/client/queries/pagination'
import {useClampedPage} from '@/composables/useClampedPage'
import {formatDateSince, formatDisplayDate} from '@/helpers/time/formatDate'
import XButton from '@/components/input/Button.vue'
import BaseButton from '@/components/base/BaseButton.vue'
import {useI18n} from 'vue-i18n'
import Message from '@/components/misc/Message.vue'
import Pagination from '@/components/misc/Pagination.vue'
import type {ApiToken as IApiToken} from '@/client/generated'
import ApiTokenForm from '@/components/token/ApiTokenForm.vue'
import {getApiBaseUrl} from '@/helpers/apiUrl'

const page = useRouteQuery('page', '1', {transform: normalizePageNumber})
const {data, isFetching, isPlaceholderData} = useQuery(computed(() => apiTokensQuery(page.value)))
useClampedPage(page, {data, isPlaceholderData})
const tokens = computed(() => data.value?.items ?? [])
const totalPages = computed(() => data.value?.total_pages ?? 0)
const deleteMutation = useDeleteApiTokenMutation()
const apiDocsUrl = `${getApiBaseUrl()}/docs`
const showCreateForm = ref(false)
const tokenCreatedSuccessMessage = ref('')

const showDeleteModal = ref<boolean>(false)
const tokenToDelete = ref<IApiToken>()

const {t} = useI18n()

const route = useRoute()

const initialTitle = ref('')
const initialScopes = ref('')

onMounted(() => {
	// Apply query parameters if present
	const titleParam = Array.isArray(route.query.title) ? route.query.title[0] : route.query.title
	const scopesParam = Array.isArray(route.query.scopes) ? route.query.scopes[0] : route.query.scopes

	if (titleParam) {
		initialTitle.value = titleParam
	}
	if (scopesParam) {
		initialScopes.value = scopesParam
	}
	if (titleParam || scopesParam) {
		showCreateForm.value = true
	}
})

async function deleteToken() {
	const token = tokenToDelete.value
	if (!token?.id) {
		return
	}
	tokenToDelete.value = undefined
	showDeleteModal.value = false
	try {
		await deleteMutation.mutateAsync(token.id)
	} catch { /* Mutation reports the error. */ }
}

function formatPermissionTitle(title: string): string {
	return title.replaceAll('_', ' ')
}

function onTokenCreated(token: IApiToken) {
	tokenCreatedSuccessMessage.value = t('user.settings.apiTokens.tokenCreatedSuccess', {token: token.token})
	showCreateForm.value = false
}
</script>

<template>
	<div class="api-tokens-container">
		<Card>
			<div class="card-header-flex">
				<div>
					<h1 class="card-title">{{ $t('user.settings.apiTokens.title') }}</h1>
					<p class="card-subtitle">
						{{ $t('user.settings.apiTokens.general') }}
						<BaseButton :href="apiDocsUrl">
							{{ $t('user.settings.apiTokens.apiDocs') }}
						</BaseButton>.
					</p>
				</div>
				<button
					v-if="!showCreateForm"
					class="btn-new-token"
					type="button"
					@click="() => showCreateForm = true"
				>
					<Icon icon="plus" />
					{{ $t('user.settings.apiTokens.createAToken') }}
				</button>
			</div>

			<Message
				v-if="tokenCreatedSuccessMessage !== ''"
				class="has-text-centered mbe-4"
			>
				{{ tokenCreatedSuccessMessage }}<br>
				{{ $t('user.settings.apiTokens.tokenCreatedNotSeeAgain') }}
			</Message>

			<div class="tokens-table-wrapper">
				<table v-if="tokens.length > 0" class="tokens-table">
					<thead>
						<tr>
							<th>{{ $t('misc.id') }}</th>
							<th>{{ $t('user.settings.apiTokens.attributes.title') }}</th>
							<th>{{ $t('user.settings.apiTokens.attributes.permissions') }}</th>
							<th>{{ $t('user.settings.apiTokens.attributes.expiresAt') }}</th>
							<th>{{ $t('misc.created') }}</th>
							<th class="has-text-end">{{ $t('misc.actions') }}</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="tk in tokens" :key="tk.id">
							<td>{{ tk.id }}</td>
							<td class="token-title">{{ tk.title }}</td>
							<td class="is-capitalized">
								<template v-for="(v, p) in tk.permissions" :key="'permission-' + p">
									<strong>{{ formatPermissionTitle(p) }}:</strong>
									{{ (v ?? []).map(formatPermissionTitle).join(', ') }}
									<br>
								</template>
							</td>
							<td>
								{{ formatDisplayDate(tk.expires_at) }}
								<p v-if="isApiTokenExpired(tk)" class="has-text-danger">
									{{ $t('user.settings.apiTokens.expired', {ago: formatDateSince(tk.expires_at)}) }}
								</p>
							</td>
							<td>{{ formatDisplayDate(tk.created) }}</td>
							<td class="has-text-end">
								<button
									type="button"
									class="btn-revoke"
									@click="() => {tokenToDelete = tk; showDeleteModal = true}"
								>
									{{ $t('misc.delete') }}
								</button>
							</td>
						</tr>
					</tbody>
				</table>
				<div v-else class="empty-state">
					Nenhum token criado até o momento.
				</div>
			</div>

			<Pagination
				:total-pages="totalPages"
				:current-page="page"
			/>

			<ApiTokenForm
				v-if="showCreateForm"
				:initial-title="initialTitle"
				:initial-scopes="initialScopes"
				@created="onTokenCreated"
				@cancel="showCreateForm = false"
			/>

			<Modal
				:enabled="showDeleteModal"
				@close="showDeleteModal = false"
				@submit="deleteToken()"
			>
				<template #header>
					{{ $t('user.settings.apiTokens.delete.header') }}
				</template>

				<template #text>
					<p v-if="tokenToDelete">
						{{ $t('user.settings.apiTokens.delete.text1', {token: tokenToDelete.title}) }}<br>
						{{ $t('user.settings.apiTokens.delete.text2') }}
					</p>
				</template>
			</Modal>
		</Card>
	</div>
</template>

<style lang="scss" scoped>
.api-tokens-container {
	/* Linear Design Tokens (Clean Light Default) */
	--font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
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
	--accent-hover: #4a55b5;
	--danger-base: #e5484d;
	--danger-bg: rgba(229, 72, 77, 0.1);
	--shadow-card: 0 1px 3px rgba(0,0,0,0.05);

	font-family: var(--font-family);
	color: var(--text-primary);
}

:global(html.dark) .api-tokens-container,
:global(body.dark-theme) .api-tokens-container {
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
	--accent-base: #5e6ad2;
	--accent-hover: #6e7be0;
	--danger-base: #ef4444;
	--danger-bg: rgba(239, 68, 68, 0.12);
	--shadow-card: 0 4px 20px rgba(0, 0, 0, 0.25);
}

.card-header-flex {
	display: flex;
	justify-content: space-between;
	align-items: flex-start;
	margin-bottom: 24px;
}

.card-title {
	font-size: 18px;
	font-weight: 600;
	letter-spacing: -0.4px;
	color: var(--text-primary);
	margin-bottom: 6px;
}

.card-subtitle {
	font-size: 13px;
	color: var(--text-secondary);
	margin-bottom: 0;
}

.btn-new-token {
	background: var(--accent-base);
	color: #ffffff;
	border: none;
	border-radius: 6px;
	padding: 8px 16px;
	font-size: 13px;
	font-weight: 600;
	cursor: pointer;
	display: inline-flex;
	align-items: center;
	gap: 6px;
	transition: background 0.15s ease;

	&:hover {
		background: var(--accent-hover);
	}
}

.tokens-table-wrapper {
	border: 1px solid var(--border-color);
	border-radius: 8px;
	overflow: hidden;
	background: var(--bg-card);
	margin-bottom: 1.5rem;
}

.tokens-table {
	width: 100%;
	border-collapse: collapse;
	font-size: 13px;

	th {
		background: var(--bg-input);
		color: var(--text-tertiary);
		font-size: 11px;
		font-weight: 600;
		text-transform: uppercase;
		letter-spacing: 0.5px;
		padding: 12px 16px;
		text-align: left;
		border-bottom: 1px solid var(--border-color);
	}

	td {
		padding: 14px 16px;
		color: var(--text-secondary);
		border-bottom: 1px solid var(--border-subtle);
		vertical-align: middle;
	}

	tr:last-child td {
		border-bottom: none;
	}

	tr:hover td {
		background: var(--bg-hover);
	}

	.token-title {
		font-weight: 500;
		color: var(--text-primary);
	}

	.btn-revoke {
		background: transparent;
		color: var(--danger-base);
		border: 1px solid var(--border-color);
		border-radius: 6px;
		padding: 4px 10px;
		font-size: 12px;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.15s ease;

		&:hover {
			background: var(--danger-bg);
			border-color: var(--danger-base);
		}
	}
}

.empty-state {
	padding: 32px 16px;
	text-align: center;
	color: var(--text-secondary);
	font-size: 13px;
}
</style>
