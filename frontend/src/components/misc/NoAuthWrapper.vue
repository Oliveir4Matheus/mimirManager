<template>
	<div class="no-auth-wrapper">
		<main
			id="main-content"
			tabindex="-1"
			class="no-auth-main"
		>
			<slot />
		</main>
	</div>
</template>

<script lang="ts" setup>
withDefaults(
	defineProps<{
		showApiConfig?: boolean
	}>(),
	{
		showApiConfig: false,
	},
)
</script>

<style lang="scss" scoped>
.no-auth-wrapper {
	min-height: 100vh;
	width: 100%;
	display: flex;
	align-items: center;
	justify-content: center;
	background-color: var(--bg-page, #f9f9f9);
	box-sizing: border-box;
}

.no-auth-main {
	display: flex;
	align-items: center;
	justify-content: center;
	width: 100%;
}
</style>
