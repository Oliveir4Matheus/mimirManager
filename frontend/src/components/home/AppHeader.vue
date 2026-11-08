<template>
	<header
		class="top-header"
		:class="{'menu-active': menuActive}"
		:style="{'--sidebar-width': sidebarWidthStyle}"
	>
		<div class="header-left">
			<MenuButton class="menu-button" />

			<div
				v-if="currentProject?.id"
				class="breadcrumb"
			>
				<span class="breadcrumb-item">MimirManager</span>
				<span class="breadcrumb-separator">/</span>
				<span class="breadcrumb-current">{{ currentProject.title === '' ? $t('misc.loading') : getProjectTitle(currentProject) }}</span>

				<BaseButton
					v-if="!isEditorContentEmpty(currentProject.description)"
					:to="{ name: 'project.info', params: { projectId: currentProject.id } }"
					class="icon-btn-inline"
					:title="$t('project.description')"
				>
					<Icon icon="circle-info" />
				</BaseButton>

				<ProjectSettingsDropdown
					v-if="canWriteCurrentProject && currentProject.id !== -1"
					class="project-title-dropdown"
					:project="currentProject"
				>
					<template #trigger="{ toggleOpen, open }">
						<BaseButton
							class="icon-btn-inline"
							:aria-expanded="open"
							@click="toggleOpen"
						>
							<Icon icon="ellipsis-h" />
						</BaseButton>
					</template>
				</ProjectSettingsDropdown>
			</div>

			<div
				v-else
				class="breadcrumb"
			>
				<span class="breadcrumb-item">MimirManager</span>
				<span class="breadcrumb-separator">/</span>
				<span class="breadcrumb-current">{{ currentSectionName }}</span>
			</div>
		</div>

		<div class="header-actions">
			<TimerBadge />
			<OpenQuickActions class="action-btn-wrapper" />
			<Notifications class="action-btn-wrapper" />
			<Dropdown class="user-dropdown">
				<template #trigger="{ toggleOpen, open }">
					<div
						class="user-profile"
						:aria-expanded="open"
						@click="toggleOpen"
					>
						<div class="avatar">
							{{ userInitials }}
						</div>
						<span class="username">{{ authStore.userDisplayName || 'Matheus' }}</span>
						<Icon
							icon="chevron-down"
							class="dropdown-icon"
							:style="{
								transform: open ? 'rotate(180deg)' : 'rotate(0)',
							}"
						/>
					</div>
				</template>

				<DropdownItem :to="{ name: 'user.settings' }">
					{{ $t('user.settings.title') }}
				</DropdownItem>
				<DropdownItem
					v-if="adminPanelEnabled && authStore.info?.is_admin"
					:to="{ name: 'admin.overview' }"
				>
					{{ $t('admin.title') }}
				</DropdownItem>
				<DropdownItem
					v-for="(link, index) in legalLinks"
					:key="index"
					:href="link.url"
				>
					{{ link.text }}
				</DropdownItem>
				<DropdownItem @click="baseStore.setKeyboardShortcutsActive(true)">
					{{ $t('keyboardShortcuts.title') }}
				</DropdownItem>
				<DropdownItem :to="{ name: 'about' }">
					{{ $t('about.title') }}
				</DropdownItem>
				<DropdownItem @click="authStore.logout()">
					{{ $t('user.auth.logout') }}
				</DropdownItem>
			</Dropdown>
		</div>
	</header>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from 'vue-i18n'

import { PERMISSIONS as Permissions } from '@/constants/permissions'
import { PRO_FEATURE } from '@/constants/proFeatures'

import ProjectSettingsDropdown from '@/components/project/ProjectSettingsDropdown.vue'
import Dropdown from '@/components/misc/Dropdown.vue'
import DropdownItem from '@/components/misc/DropdownItem.vue'
import Notifications from '@/components/notifications/Notifications.vue'
import TimerBadge from '@/components/time-tracking/TimerBadge.vue'
import BaseButton from '@/components/base/BaseButton.vue'
import MenuButton from '@/components/home/MenuButton.vue'
import OpenQuickActions from '@/components/misc/OpenQuickActions.vue'

import { getProjectTitle } from '@/helpers/getProjectTitle'
import { isEditorContentEmpty } from '@/helpers/editorContentEmpty'

import { useBaseStore } from '@/stores/base'
import { useConfigStore } from '@/stores/config'
import { useAuthStore } from '@/stores/auth'
import { useCurrentProject } from '@/composables/useCurrentProject'
import { useLegalLinks } from '@/composables/useLegalLinks'
import { useSidebarResize } from '@/composables/useSidebarResize'

const baseStore = useBaseStore()
const { currentProject } = useCurrentProject()
const { sidebarWidthStyle } = useSidebarResize()
const canWriteCurrentProject = computed(() =>
	currentProject.value?.max_permission !== null &&
	currentProject.value?.max_permission !== undefined &&
	currentProject.value.max_permission > Permissions.READ,
)
const menuActive = computed(() => baseStore.menuActive)

const route = useRoute()
const { t } = useI18n()

const currentSectionName = computed(() => {
	const name = route.name as string | undefined
	if (!name) return 'Overview'
	if (name === 'home') return 'Overview'
	if (name === 'inbox') return 'Inbox'
	if (name === 'tasks.range') return 'Upcoming'
	if (name === 'projects.index') return 'Projects'
	if (name === 'project.new') return 'Novo Projeto'
	if (name === 'labels.index') return 'Labels'
	if (name === 'teams.index') return 'Teams'
	if (name === 'teams.new') return 'Criar Equipe'
	if (name === 'teams.edit') return 'Editar Equipe'
	if (name === 'trash' || name.startsWith('trash.')) return 'Trash'
	if (name === 'time-tracking') return 'Time Tracking'
	if (name.startsWith('user.settings')) return t('user.settings.title') || 'Configurações'
	if (name.startsWith('admin.')) return t('admin.title') || 'Administração'
	if (name === 'about') return t('about.title') || 'Sobre'
	const title = route.meta.title as string | undefined
	return title ? t(title) : 'Overview'
})

const authStore = useAuthStore()
const userInitials = computed(() => {
	const name = authStore.userDisplayName || 'Matheus'
	return name.charAt(0).toUpperCase()
})

const configStore = useConfigStore()
const legalLinks = useLegalLinks()
const adminPanelEnabled = computed(() => configStore.isProFeatureEnabled(PRO_FEATURE.ADMIN_PANEL))
</script>

<style lang="scss" scoped>
.top-header {
	height: 48px;
	position: fixed;
	inset-block-start: 0;
	inset-inline-start: var(--sidebar-width, 240px);
	inset-inline-end: 0;
	z-index: 30;
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 0 24px;
	border-bottom: 1px solid var(--border-color);
	background: var(--bg-app);
	transition: inset-inline-start $transition-duration;

	@media screen and (max-width: $tablet) {
		inset-inline-start: 0;
		padding: 0 16px;
	}
}

.header-left {
	display: flex;
	align-items: center;
	gap: 12px;
	min-width: 0;
}

.menu-button {
	display: none;

	@media screen and (max-width: $tablet) {
		display: flex;
	}
}

.breadcrumb {
	display: flex;
	align-items: center;
	gap: 8px;
	font-size: 14px;
	font-weight: 600;
	color: var(--text-primary);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;

	.breadcrumb-item {
		color: var(--text-secondary);
		font-weight: 500;
	}

	.breadcrumb-separator {
		color: var(--text-tertiary);
		font-weight: 400;
	}

	.breadcrumb-current {
		color: var(--text-primary);
		font-weight: 600;
		overflow: hidden;
		text-overflow: ellipsis;
	}
}

.icon-btn-inline {
	background: transparent;
	border: none;
	cursor: pointer;
	color: var(--text-secondary);
	padding: 2px 4px;
	border-radius: 4px;
	display: inline-flex;
	align-items: center;

	&:hover {
		background: var(--bg-hover);
		color: var(--text-primary);
	}
}

.project-title-dropdown {
	display: inline-flex;
	align-items: center;
}

.header-actions {
	display: flex;
	align-items: center;
	gap: 8px;
	flex-shrink: 0;

	:deep(.trigger-button) {
		background: transparent;
		border: none;
		cursor: pointer;
		color: var(--text-secondary);
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 6px;
		border-radius: 6px;
		width: 32px;
		height: 32px;
		font-size: 14px;
		transition: background 0.15s, color 0.15s;

		&:hover {
			background: var(--bg-hover);
			color: var(--text-primary);
		}
	}
}

.user-profile {
	display: flex;
	align-items: center;
	gap: 8px;
	padding: 3px 8px;
	border-radius: 6px;
	cursor: pointer;
	font-size: 13px;
	font-weight: 500;
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
		flex-shrink: 0;
	}

	.username {
		color: var(--text-primary);
		white-space: nowrap;

		@media screen and (max-width: $tablet) {
			display: none;
		}
	}

	.dropdown-icon {
		font-size: 11px;
		color: var(--text-tertiary);
		transition: transform 0.2s ease;
	}
}
</style>
