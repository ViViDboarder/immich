<script lang="ts">
	import { AlbumResponseDto, api, ThumbnailFormat, UserResponseDto } from '@api';
	import { fade } from 'svelte/transition';

	export let album: AlbumResponseDto;
	export let user: UserResponseDto;

	const loadImageData = async (thubmnailId: string | null) => {
		if (thubmnailId == null) {
			return '/no-thumbnail.png';
		}

		const { data } = await api.assetApi.getAssetThumbnail(thubmnailId, ThumbnailFormat.Webp, {
			responseType: 'blob'
		});
		if (data instanceof Blob) {
			return URL.createObjectURL(data);
		}
	};

	const getAlbumOwnerInfo = async (): Promise<UserResponseDto> => {
		const { data } = await api.userApi.getUserById(album.ownerId);

		return data;
	};
</script>

<div
	class="flex min-w-[550px] border-b border-gray-300 dark:border-immich-dark-gray place-items-center py-4  gap-6 transition-all hover:border-immich-primary dark:hover:border-immich-dark-primary"
>
	<div>
		{#await loadImageData(album.albumThumbnailAssetId)}
			<div
				class={`bg-immich-primary/10 w-[75px] h-[75px] flex place-items-center place-content-center rounded-xl`}
			>
				...
			</div>
		{:then imageData}
			<img
				in:fade={{ duration: 250 }}
				src={imageData}
				alt={album.id}
				class={`object-cover w-[75px] h-[75px] transition-all z-0 rounded-xl duration-300 `}
			/>
		{/await}
	</div>

	<div>
		<p class="font-medium text-gray-800 dark:text-immich-dark-primary">{album.albumName}</p>

		{#await getAlbumOwnerInfo() then albumOwner}
			{#if user.email == albumOwner.email}
				<p class="text-xs text-gray-600 dark:text-immich-dark-fg">Owned</p>
			{:else}
				<p class="text-xs text-gray-600 dark:text-immich-dark-fg">
					Shared by {albumOwner.firstName}
					{albumOwner.lastName}
				</p>
			{/if}
		{/await}
	</div>
</div>
