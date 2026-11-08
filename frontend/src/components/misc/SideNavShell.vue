<template>
	<div class="content-widescreen">
		<div class="side-nav-shell">
			<nav class="navigation">
				<ul>
					<li
						v-for="(item, index) in navigationItems"
						:key="`nav-${index}`"
					>
						<RouterLink
							v-slot="{href, navigate, isActive, isExactActive}"
							:to="{name: item.routeName}"
							custom
						>
							<a
								:href="href"
								class="navigation-link"
								:class="{'is-active': (exact ? isExactActive : isActive) || isAliasActive(item)}"
								@click="navigate"
							>
								{{ item.title }}
							</a>
						</RouterLink>
					</li>
					<li
						v-for="({url, text}, index) in extraLinks"
						:key="`extra-${index}`"
					>
						<BaseButton
							class="navigation-link is-flex is-align-items-center"
							:href="url"
						>
							<span>
								{{ text }}
							</span>
							<span class="ml-1 has-text-grey-light is-size-7">
								<Icon
									icon="arrow-up-right-from-square"
								/>
							</span>
						</BaseButton>
					</li>
				</ul>
			</nav>
			<section class="view">
				<RouterView />
			</section>
		</div>
	</div>
</template>

<script setup lang="ts">
import {useRoute} from 'vue-router'

import BaseButton from '@/components/base/BaseButton.vue'

export interface SideNavItem {
	title: string
	routeName: string
	activeRouteNames?: string[]
}

export interface SideNavExtraLink {
	url: string
	text: string
}

withDefaults(defineProps<{
	navigationItems: SideNavItem[]
	extraLinks?: SideNavExtraLink[]
	exact?: boolean
}>(), {
	extraLinks: () => [],
	exact: false,
})

const route = useRoute()

function isAliasActive(item: SideNavItem) {
	return item.activeRouteNames?.includes(route.name as string) ?? false
}
</script>

<style lang="scss" scoped>
.side-nav-shell {
	/* Design Tokens */
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
	--shadow-card: 0 1px 3px rgba(0,0,0,0.05);

	display: flex;
	font-family: var(--font-family);
	color: var(--text-primary);
	gap: 24px;
	padding-top: 1rem;

	@media screen and (max-width: $tablet) {
		flex-direction: column;
	}
}

:global(html.dark) .side-nav-shell,
:global(body.dark-theme) .side-nav-shell {
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
	--shadow-card: 0 4px 20px rgba(0, 0, 0, 0.25);
}

.navigation {
	width: 240px;
	min-width: 240px;
	flex-shrink: 0;

	@media screen and (max-width: $tablet) {
		width: 100%;
		min-width: 100%;
		padding-inline-end: 0;
	}

	ul {
		list-style: none;
		margin: 0;
		padding: 0;
		display: flex;
		flex-direction: column;
		gap: 2px;
	}
}

.navigation-link {
	display: flex;
	align-items: center;
	padding: 8px 12px;
	color: var(--text-secondary);
	font-size: 13px;
	font-weight: 500;
	border-radius: 6px;
	text-decoration: none;
	border-inline-start: 2px solid transparent;
	transition: background 0.15s, color 0.15s;

	&:hover {
		background: var(--bg-hover);
		color: var(--text-primary);
	}

	&.is-active {
		background: var(--bg-hover);
		color: var(--text-primary);
		font-weight: 600;
		border-inline-start: 2px solid var(--accent-base);
	}
}

.view {
	flex: 1;
	min-width: 0;

	@media screen and (max-width: $tablet) {
		width: 100%;
		padding-block-start: 1rem;
	}
}
</style>
