<script lang="ts">
	import { goto } from '$app/navigation';
	import type { PageData } from './$types';

	export let data: PageData;

	async function onGettingStartedClicked() {
		data.isAdminUserExist ? await goto('/auth/login') : await goto('/auth/register');
	}
</script>

<svelte:head>
	<title>Welcome 🎉 - Immich</title>
	<meta name="description" content="Immich Web Interface" />
</svelte:head>

<section class="h-screen w-screen flex place-items-center place-content-center">
	<div class="flex flex-col place-items-center gap-8 text-center max-w-[350px]">
		<div class="flex place-items-center place-content-center ">
			<img class="text-center" src="immich-logo.svg" height="200" width="200" alt="immich-logo" />
		</div>
		<h1
			class="text-4xl text-immich-primary dark:text-immich-dark-primary font-bold font-immich-title"
		>
			Welcome to IMMICH Web
		</h1>
		<button
			class="border px-4 py-4 rounded-md bg-immich-primary dark:bg-immich-dark-primary dark:text-immich-dark-gray dark:border-immich-dark-gray hover:bg-immich-primary/75 text-white font-bold w-[200px]"
			on:click={onGettingStartedClicked}
			>Getting Started
		</button>
	</div>
</section>
