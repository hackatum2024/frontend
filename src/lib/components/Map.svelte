<!-- src/lib/components/Map.svelte -->
<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';

	// Props
	export let selectedLocation: [number, number] = [48.137154, 11.576124];
	export let zoom: number = 13;

	// Define types but don't initialize yet
	let map: any;
	let marker: any;
	let L: any;
	let mapElement: HTMLDivElement;

	onMount(async () => {
		if (!browser) return;

		// Dynamically import Leaflet only on client side
		const leaflet = await import('leaflet');
		L = leaflet.default;

		// Initialize map after Leaflet is loaded
		map = L.map(mapElement).setView(selectedLocation, zoom);

		// Add OpenStreetMap tiles
		L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
			attribution:
				'&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
		}).addTo(map);

		// Custom car marker icon
		const carIcon = L.divIcon({
			html: `<div class="car-marker">🚗</div>`,
			className: 'car-marker-container',
			iconSize: [24, 24],
			iconAnchor: [12, 12]
		});

		// Add initial marker
		marker = L.marker(selectedLocation, { icon: carIcon }).addTo(map);

		// Disable scroll zoom
		map.scrollWheelZoom.disable();
	});

	// Update marker position when selectedLocation changes
	$: if (map && marker && selectedLocation) {
		marker.setLatLng(selectedLocation);
		map.setView(selectedLocation, zoom, {
			animate: true,
			duration: 0.5
		});
	}

	onDestroy(() => {
		if (map) {
			map.remove();
		}
	});
</script>

<div class="map-container" bind:this={mapElement}>
	<!-- Map will be initialized here -->
</div>

<style>
	.map-container {
		width: 100%;
		height: 100%;
		z-index: 1;
	}

	:global(.car-marker-container) {
		background: none;
		border: none;
	}

	:global(.car-marker) {
		font-size: 24px;
		display: flex;
		align-items: center;
		justify-content: center;
		animation: bounce 0.3s ease-out;
	}

	@keyframes bounce {
		0% {
			transform: translateY(-20px);
			opacity: 0;
		}
		100% {
			transform: translateY(0);
			opacity: 1;
		}
	}

	/* Override some Leaflet default styles */
	:global(.leaflet-control-container .leaflet-control) {
		margin: 10px;
	}

	:global(.leaflet-control-zoom) {
		border: none !important;
		border-radius: 8px !important;
		overflow: hidden;
	}

	:global(.leaflet-control-zoom a) {
		background-color: white !important;
		color: #333 !important;
		width: 30px !important;
		height: 30px !important;
		line-height: 30px !important;
	}
</style>

