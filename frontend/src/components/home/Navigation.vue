<template>
	<aside
		:class="{'is-active': baseStore.menuActive, 'is-resizing': isResizing}"
		class="sidebar menu-container"
		:style="{'--sidebar-width': sidebarWidthStyle}"
	>
		<div class="sidebar-header">
			<RouterLink
				:to="{name: 'home'}"
				class="sidebar-brand"
				:aria-label="$t('navigation.home')"
				@click="handleNavClick"
			>
				<div class="logo-placeholder">V</div>
				<span class="app-name">MimirManager</span>
			</RouterLink>
		</div>

		<div class="nav-menu">
			<RouterLink
				v-shortcut="SHORTCUTS.navigation.overview"
				:to="{ name: 'home'}"
				class="nav-item"
				:class="{ active: $route.name === 'home' }"
				@click="handleNavClick"
			>
				<svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
				</svg>
				<span>{{ $t('navigation.overview') }}</span>
			</RouterLink>

			<RouterLink
				v-shortcut="SHORTCUTS.navigation.upcoming"
				:to="{ name: 'tasks.range'}"
				class="nav-item"
				:class="{ active: $route.name === 'tasks.range' }"
				@click="handleNavClick"
			>
				<svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
					<line x1="16" y1="2" x2="16" y2="6"></line>
					<line x1="8" y1="2" x2="8" y2="6"></line>
					<line x1="3" y1="10" x2="21" y2="10"></line>
				</svg>
				<span>{{ $t('navigation.upcoming') }}</span>
			</RouterLink>

			<RouterLink
				v-shortcut="SHORTCUTS.navigation.projects"
				:to="{ name: 'projects.index'}"
				class="nav-item"
				:class="{ active: $route.name === 'projects.index' || $route.name === 'project.new' }"
				@click="handleNavClick"
			>
				<svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"></path>
				</svg>
				<span>{{ $t('project.projects') }}</span>
			</RouterLink>

			<RouterLink
				v-shortcut="SHORTCUTS.navigation.labels"
				:to="{ name: 'labels.index'}"
				class="nav-item"
				:class="{ active: $route.name === 'labels.index' }"
				@click="handleNavClick"
			>
				<svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<path d="M20.59 13.41l-7.17 7.17a2 2 0 0 1-2.83 0L2 12V2h10l8.59 8.59a2 2 0 0 1 0 2.82z"></path>
					<line x1="7" y1="7" x2="7.01" y2="7"></line>
				</svg>
				<span>{{ $t('label.title') }}</span>
			</RouterLink>

			<RouterLink
				v-shortcut="SHORTCUTS.navigation.teams"
				:to="{ name: 'teams.index'}"
				class="nav-item"
				:class="{ active: $route.name === 'teams.index' || $route.name === 'teams.new' || $route.name === 'teams.edit' }"
				@click="handleNavClick"
			>
				<svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
					<circle cx="9" cy="7" r="4"></circle>
					<path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
					<path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
				</svg>
				<span>{{ $t('team.title') }}</span>
			</RouterLink>

			<RouterLink
				:to="{ name: 'trash'}"
				class="nav-item"
				:class="{ active: $route.name === 'trash' || $route.name === 'trash.tasks' || $route.name === 'trash.projects' }"
				@click="handleNavClick"
			>
				<svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<polyline points="3 6 5 6 21 6"></polyline>
					<path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
				</svg>
				<span>{{ $t('trash.title') }}</span>
			</RouterLink>

			<div class="nav-section-title">
				{{ $t('tasks.myOpenTasks') || 'Minhas Tarefas' }}
			</div>

			<RouterLink
				:to="{ name: 'inbox'}"
				class="nav-item"
				:class="{ active: $route.name === 'inbox' }"
				@click="handleNavClick"
			>
				<svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<polyline points="22 12 16 12 14 15 10 15 8 12 2 12"></polyline>
					<path d="M5.45 5.11L2 12v6a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-6l-3.45-6.89A2 2 0 0 0 16.76 4H7.24a2 2 0 0 0-1.79 1.11z"></path>
				</svg>
				<span>{{ $t('project.inboxTitle') }}</span>
			</RouterLink>

			<RouterLink
				v-if="timeTrackingEnabled"
				:to="{ name: 'time-tracking'}"
				class="nav-item"
				:class="{ active: $route.name === 'time-tracking' }"
				@click="handleNavClick"
			>
				<svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<circle cx="12" cy="12" r="10"></circle>
					<polyline points="12 6 12 12 16 14"></polyline>
				</svg>
				<span>{{ $t('timeTracking.title') }}</span>
			</RouterLink>

			<Loading
				v-if="projectList.isLoading"
				variant="small"
				class="p-4"
			/>
			<template v-else>
				<div
					v-if="favoriteProjects.length"
					class="projects-section"
				>
					<div class="nav-section-title">
						{{ $t('project.pseudo.favorites.title') }}
					</div>
					<ProjectsNavigation
						:model-value="favoriteProjects"
						:can-edit-order="false"
						:can-collapse="false"
					/>
				</div>

				<div
					v-if="savedFilterProjects.length"
					class="projects-section"
				>
					<div class="nav-section-title">
						{{ $t('navigation.savedFilters') }}
					</div>
					<ProjectsNavigation
						:model-value="savedFilterProjects"
						:can-edit-order="false"
						:can-collapse="false"
					/>
				</div>

				<div
					v-if="projects.length"
					class="projects-section"
				>
					<div class="nav-section-title">
						{{ $t('project.projects') }}
					</div>
					<ProjectsNavigation
						:model-value="projects"
						:can-edit-order="true"
						:can-collapse="true"
					/>
				</div>
			</template>
		</div>

		<div class="sidebar-footer">
			<PoweredByLink utm-medium="navigation" />
		</div>

		<div
			v-if="!isMobile"
			class="resize-handle"
			@mousedown="startResize"
			@touchstart="startResize"
		/>
	</aside>
</template>

<script setup lang="ts">
import {computed} from 'vue'

import {SHORTCUTS} from '@/constants/shortcuts'
import PoweredByLink from '@/components/home/PoweredByLink.vue'
import Loading from '@/components/misc/Loading.vue'

import {useBaseStore} from '@/stores/base'
import {useProjects} from '@/composables/useProjects'
import {useConfigStore} from '@/stores/config'
import {PRO_FEATURE} from '@/constants/proFeatures'
import ProjectsNavigation from '@/components/home/ProjectsNavigation.vue'
import {useSidebarResize} from '@/composables/useSidebarResize'

const baseStore = useBaseStore()
const projectList = useProjects()
const configStore = useConfigStore()

const timeTrackingEnabled = computed(() => configStore.isProFeatureEnabled(PRO_FEATURE.TIME_TRACKING))

const {sidebarWidthStyle, isResizing, startResize, isMobile} = useSidebarResize()

const projects = computed(() => projectList.notArchivedRootProjects)
const favoriteProjects = computed(() => projectList.favoriteProjects)
const savedFilterProjects = computed(() => projectList.savedFilterProjects)

function handleNavClick() {
	if (isMobile.value) {
		baseStore.setMenuActive(false)
	}
}
</script>

<style lang="scss" scoped>
.sidebar {
	--sidebar-width: 240px;

	position: fixed;
	inset-block-start: 0;
	inset-block-end: 0;
	inset-inline-start: 0;
	inline-size: var(--sidebar-width);
	height: 100vh;
	background: var(--bg-sidebar);
	border-right: 1px solid var(--border-color);
	display: flex;
	flex-direction: column;
	flex-shrink: 0;
	z-index: 35;
	transition: transform $transition-duration ease-in;
	overflow: hidden;

	[dir="rtl"] & {
		border-right: none;
		border-left: 1px solid var(--border-color);
	}

	@media screen and (max-width: $tablet) {
		transform: translateX(-100%);
		inline-size: 260px;
		z-index: 40;

		[dir="rtl"] & {
			transform: translateX(100%);
		}

		&.is-active {
			transform: translateX(0);
			transition: transform $transition-duration ease-out;
		}
	}

	&.is-resizing {
		transition: none;
	}
}

.sidebar-header {
	height: 48px;
	padding: 0 16px;
	display: flex;
	align-items: center;
	border-bottom: 1px solid var(--border-subtle);
	flex-shrink: 0;
}

.sidebar-brand {
	display: flex;
	align-items: center;
	gap: 12px;
	text-decoration: none;
	color: inherit;
}

.logo-placeholder {
	width: 24px;
	height: 24px;
	background: var(--accent-base);
	border-radius: 6px;
	display: flex;
	align-items: center;
	justify-content: center;
	color: #ffffff;
	font-weight: 700;
	font-size: 12px;
	flex-shrink: 0;
}

.app-name {
	font-size: 14px;
	font-weight: 600;
	letter-spacing: -0.3px;
	color: var(--text-primary);
}

.nav-menu {
	padding: 12px;
	flex: 1;
	overflow-y: auto;
}

.nav-item {
	display: flex;
	align-items: center;
	gap: 12px;
	padding: 6px 8px;
	font-size: 13px;
	font-weight: 500;
	color: var(--text-secondary);
	border-radius: 6px;
	cursor: pointer;
	margin-bottom: 2px;
	text-decoration: none;
	transition: background 0.1s, color 0.1s;

	&:hover {
		background: var(--bg-hover);
		color: var(--text-primary);

		.nav-icon {
			color: var(--text-primary);
		}
	}

	&.active,
	&.router-link-exact-active {
		background: var(--bg-hover);
		color: var(--text-primary);
		font-weight: 600;

		.nav-icon {
			color: var(--accent-base);
		}
	}
}

.nav-icon {
	width: 16px;
	height: 16px;
	color: var(--text-tertiary);
	flex-shrink: 0;
	transition: color 0.1s;
}

.nav-section-title {
	font-size: 11px;
	text-transform: uppercase;
	font-weight: 600;
	letter-spacing: 0.5px;
	color: var(--text-tertiary);
	padding: 16px 8px 8px 8px;
	margin-top: 8px;
}

.projects-section {
	margin-top: 4px;

	:deep(.menu-list) {
		padding: 0;
		margin: 0;
		list-style: none;

		li {
			height: auto;
		}

		a {
			padding: 5px 8px;
			font-size: 13px;
			color: var(--text-secondary);
			border-radius: 6px;
			display: flex;
			align-items: center;
			text-decoration: none;
			transition: background 0.1s, color 0.1s;

			&:hover {
				background: var(--bg-hover);
				color: var(--text-primary);
			}

			&.router-link-exact-active {
				background: var(--bg-hover);
				color: var(--text-primary);
				font-weight: 600;
			}
		}
	}
}

.sidebar-footer {
	padding: 12px 16px;
	border-top: 1px solid var(--border-subtle);
	flex-shrink: 0;
	font-size: 11px;
	color: var(--text-tertiary);
}

.resize-handle {
	position: absolute;
	inset-block-start: 0;
	inset-block-end: 0;
	inset-inline-end: 0;
	inline-size: 4px;
	cursor: ew-resize;
	background: transparent;
	transition: background-color $transition-duration ease;
	touch-action: none;

	&:hover,
	&:active {
		background-color: var(--accent-base);
	}
}
</style>
