<script>
	import "leaflet/dist/leaflet.css";
	import "../../map.css";
	import { fly } from "svelte/transition";
	import Map from "$components/map/Map.svelte";
	import SearchPanel from "$components/ui/panels/SearchPanel.svelte";
	import UserPanel from "$components/ui/panels/UserPanel.svelte";
	import InfoPanel from "$components/ui/panels/InfoPanel.svelte";
	import { mapData, showVideo, activePanel, waypoint, language } from "$store";
	import { mobileSize } from "$scripts/media-queries";
	import { maps } from "$data/maps";
	import { keys } from "$data/keys";
	import { page } from "$app/stores";

	let outerWidth;
	let translatedKeys = keys;

	function showMapToggle(panelIsOpen, outerWidth) {
		if (panelIsOpen && mobileSize(outerWidth)) {
			return false;
		}
		if (Object.keys(maps).length > 1) return true;
		return false;
	}

	$: setMapData($page.params?.map);

	function setMapData(map) {
		if ($mapData?.title !== maps[map]?.title) {
			// Change map data
			$mapData = maps[map];
			$waypoint = null;
		}
	}

	function translateKeys(locale) {
		if (locale && locale !== "en") {
			translatedKeys = keys.map((key) => {
				return {
					...key,
					...key.locales[locale],
				};
			});
		} else {
			translatedKeys = keys;
		}
	}

	language.subscribe((locale) => translateKeys(locale));
</script>

<svelte:window bind:outerWidth />

{#if showMapToggle($activePanel, outerWidth)}
	<nav transition:fly={{ y: -40 }}>
		<ul>
			{#each Object.keys(maps) as map}
				<li class:active={map === $page?.params?.map}>
					<a href="../{map}">{maps[map]?.title}</a>
				</li>
			{/each}
		</ul>
	</nav>
{/if}
<main class:video-disabled={!$showVideo}>
	<UserPanel />
	<InfoPanel />
	<SearchPanel keys={translatedKeys} />
	<Map keys={translatedKeys} />
	<slot />
</main>

<style>
	nav {
		position: fixed;
		z-index: 100000;
		left: 50%;
		top: 10px;
		transform: translate(-50%);
		border-radius: 0.5rem;
		background-color: var(--color-black);
		box-shadow: 0 0 1rem rgba(0, 0, 0, 0.2);
		opacity: 0.9;
		overflow: hidden;
	}

	ul {
		display: flex;
		list-style: none;
		margin: 0;
		padding: 0;
		text-align: center;
		width: 100%;
	}

	a {
		display: inline-block;
		padding: 0.5rem 1rem;
		text-decoration: none;
	}

	li a {
		opacity: 0.75;
	}

	li.active a {
		opacity: 1;
		font-weight: bold;
		cursor: default;
		pointer-events: none;
		background-color: var(--color-black-light);
	}
</style>
