<script lang="ts">
	import { createEventDispatcher, onMount, onDestroy } from 'svelte';
	import { fly } from 'svelte/transition';
	import AsserViewerNavBar from './asset-viewer-nav-bar.svelte';
	import ChevronRight from 'svelte-material-icons/ChevronRight.svelte';
	import ChevronLeft from 'svelte-material-icons/ChevronLeft.svelte';
	import PhotoViewer from './photo-viewer.svelte';
	import DetailPanel from './detail-panel.svelte';
	import { goto } from '$app/navigation';
	import { downloadAssets } from '$lib/stores/download';
	import VideoViewer from './video-viewer.svelte';
	import AlbumSelectionModal from './album-selection-modal.svelte';
	import {
		api,
		AddAssetsResponseDto,
		AssetResponseDto,
		AssetTypeEnum,
		AlbumResponseDto
	} from '@api';
	import {
		notificationController,
		NotificationType
	} from '../shared-components/notification/notification';
	import { assetStore } from '$lib/stores/assets.store';

	export let asset: AssetResponseDto;
	$: {
		appearsInAlbums = [];

		api.albumApi.getAllAlbums(undefined, asset.id).then((result) => {
			appearsInAlbums = result.data;
		});
	}

	const dispatch = createEventDispatcher();
	let halfLeftHover = false;
	let halfRightHover = false;
	let isShowDetail = false;
	let appearsInAlbums: AlbumResponseDto[] = [];
	let isShowAlbumPicker = false;
	let addToSharedAlbum = true;

	const onKeyboardPress = (keyInfo: KeyboardEvent) => handleKeyboardPress(keyInfo.key);

	onMount(() => {
		document.addEventListener('keydown', onKeyboardPress);
	});

	onDestroy(() => {
		document.removeEventListener('keydown', onKeyboardPress);
	});

	const handleKeyboardPress = (key: string) => {
		switch (key) {
			case 'Escape':
				closeViewer();
				return;
			case 'Delete':
				deleteAsset();
				return;
			case 'i':
				isShowDetail = !isShowDetail;
				return;
			case 'ArrowLeft':
				navigateAssetBackward();
				return;
			case 'ArrowRight':
				navigateAssetForward();
				return;
		}
	};

	const closeViewer = () => {
		dispatch('close');
	};

	const navigateAssetForward = (e?: Event) => {
		e?.stopPropagation();
		dispatch('navigate-next');
	};

	const navigateAssetBackward = (e?: Event) => {
		e?.stopPropagation();
		dispatch('navigate-previous');
	};

	const showDetailInfoHandler = () => {
		isShowDetail = !isShowDetail;
	};

	const downloadFile = async () => {
		try {
			const imageName = asset.exifInfo?.imageName ? asset.exifInfo?.imageName : asset.id;
			const imageExtension = asset.originalPath.split('.')[1];
			const imageFileName = imageName + '.' + imageExtension;

			// If assets is already download -> return;
			if ($downloadAssets[imageFileName]) {
				return;
			}

			$downloadAssets[imageFileName] = 0;

			const { data, status } = await api.assetApi.downloadFile(
				asset.deviceAssetId,
				asset.deviceId,
				false,
				false,
				{
					responseType: 'blob',
					onDownloadProgress: (progressEvent) => {
						if (progressEvent.lengthComputable) {
							const total = progressEvent.total;
							const current = progressEvent.loaded;
							$downloadAssets[imageFileName] = Math.floor((current / total) * 100);
						}
					}
				}
			);

			if (!(data instanceof Blob)) {
				return;
			}

			if (status === 200) {
				const fileUrl = URL.createObjectURL(data);
				const anchor = document.createElement('a');
				anchor.href = fileUrl;
				anchor.download = imageFileName;

				document.body.appendChild(anchor);
				anchor.click();
				document.body.removeChild(anchor);

				URL.revokeObjectURL(fileUrl);

				// Remove item from download list
				setTimeout(() => {
					const copy = $downloadAssets;
					delete copy[imageFileName];
					$downloadAssets = copy;
				}, 2000);
			}
		} catch (e) {
			console.error('Error downloading file ', e);
			notificationController.show({
				type: NotificationType.Error,
				message: 'Error downloading file, check console for more details.'
			});
		}
	};

	const deleteAsset = async () => {
		try {
			if (
				window.confirm(
					`Caution! Are you sure you want to delete this asset? This step also deletes this asset in the album(s) to which it belongs. You can not undo this action!`
				)
			) {
				const { data: deletedAssets } = await api.assetApi.deleteAsset({
					ids: [asset.id]
				});

				navigateAssetForward();

				for (const asset of deletedAssets) {
					if (asset.status == 'SUCCESS') {
						assetStore.removeAsset(asset.id);
					}
				}
			}
		} catch (e) {
			notificationController.show({
				type: NotificationType.Error,
				message: 'Error deleting this asset, check console for more details'
			});
			console.error('Error deleteSelectedAssetHandler', e);
		}
	};

	const openAlbumPicker = (shared: boolean) => {
		isShowAlbumPicker = true;
		addToSharedAlbum = shared;
	};

	const showAddNotification = (dto: AddAssetsResponseDto) => {
		notificationController.show({
			message: `Added ${dto.successfullyAdded} to ${dto.album?.albumName}`,
			type: NotificationType.Info
		});

		if (dto.successfullyAdded === 1 && dto.album) {
			appearsInAlbums = [...appearsInAlbums, dto.album];
		}
	};

	const handleAddToNewAlbum = () => {
		isShowAlbumPicker = false;
		api.albumApi.createAlbum({ albumName: 'Untitled', assetIds: [asset.id] }).then((response) => {
			const album = response.data;
			goto('/albums/' + album.id);
		});
	};

	const handleAddToAlbum = async (event: CustomEvent<{ album: AlbumResponseDto }>) => {
		isShowAlbumPicker = false;
		const album = event.detail.album;

		api.albumApi
			.addAssetsToAlbum(album.id, { assetIds: [asset.id] })
			.then((response) => showAddNotification(response.data));
	};
</script>

<section
	id="immich-asset-viewer"
	class="fixed h-screen w-screen top-0 overflow-y-hidden bg-black z-[999] grid grid-rows-[64px_1fr] grid-cols-4"
>
	<div class="col-start-1 col-span-4 row-start-1 row-span-1 z-[1000] transition-transform">
		<AsserViewerNavBar
			on:goBack={closeViewer}
			on:showDetail={showDetailInfoHandler}
			on:download={downloadFile}
			on:delete={deleteAsset}
			on:addToAlbum={() => openAlbumPicker(false)}
			on:addToSharedAlbum={() => openAlbumPicker(true)}
		/>
	</div>

	<div
		class={`row-start-2 row-span-end col-start-1 col-span-2 flex place-items-center hover:cursor-pointer w-3/4 mb-[60px] ${
			asset.type === AssetTypeEnum.Video ? '' : 'z-[999]'
		}`}
		on:mouseenter={() => {
			halfLeftHover = true;
			halfRightHover = false;
		}}
		on:mouseleave={() => {
			halfLeftHover = false;
		}}
		on:click={navigateAssetBackward}
	>
		<button
			class="rounded-full p-3 hover:bg-gray-500 hover:text-gray-700 z-[1000]  text-gray-500 mx-4"
			class:navigation-button-hover={halfLeftHover}
			on:click={navigateAssetBackward}
		>
			<ChevronLeft size="36" />
		</button>
	</div>

	<div class="row-start-1 row-span-full col-start-1 col-span-4">
		{#key asset.id}
			{#if asset.type === AssetTypeEnum.Image}
				<PhotoViewer assetId={asset.id} deviceId={asset.deviceId} on:close={closeViewer} />
			{:else}
				<VideoViewer assetId={asset.id} on:close={closeViewer} />
			{/if}
		{/key}
	</div>

	<div
		class={`row-start-2 row-span-full col-start-3 col-span-2 flex justify-end place-items-center hover:cursor-pointer w-3/4 justify-self-end mb-[60px] ${
			asset.type === AssetTypeEnum.Video ? '' : 'z-[500]'
		}`}
		on:click={navigateAssetForward}
		on:mouseenter={() => {
			halfLeftHover = false;
			halfRightHover = true;
		}}
		on:mouseleave={() => {
			halfRightHover = false;
		}}
	>
		<button
			class="rounded-full p-3 hover:bg-gray-500 hover:text-gray-700 text-gray-500 mx-4"
			class:navigation-button-hover={halfRightHover}
			on:click={navigateAssetForward}
		>
			<ChevronRight size="36" />
		</button>
	</div>

	{#if isShowDetail}
		<div
			transition:fly={{ duration: 150 }}
			id="detail-panel"
			class="bg-immich-bg w-[360px] row-span-full transition-all overflow-y-auto dark:bg-immich-dark-bg dark:border-l dark:border-l-immich-dark-gray"
			translate="yes"
		>
			<DetailPanel {asset} albums={appearsInAlbums} on:close={() => (isShowDetail = false)} />
		</div>
	{/if}

	{#if isShowAlbumPicker}
		<AlbumSelectionModal
			shared={addToSharedAlbum}
			on:newAlbum={handleAddToNewAlbum}
			on:newSharedAlbum={handleAddToNewAlbum}
			on:album={handleAddToAlbum}
			on:close={() => (isShowAlbumPicker = false)}
		/>
		<div class="w-full h-full">Hello</div>
	{/if}
</section>

<style>
	#immich-asset-viewer {
		contain: layout;
	}

	.navigation-button-hover {
		background-color: rgb(107 114 128 / var(--tw-bg-opacity));
		color: rgb(55 65 81 / var(--tw-text-opacity));
		transition: all 150ms;
	}
</style>
