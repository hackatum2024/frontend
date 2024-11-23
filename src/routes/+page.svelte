<!-- src/routes/+page.svelte -->
<script lang="ts">
	import Map from '$lib/components/Map.svelte';
	import { onMount, tick } from 'svelte';
	import { spring } from 'svelte/motion';

	// Mock car data with fixed map positions
	const carData = [
		{
			id: 1,
			name: 'VW Golf',
			price: 45,
			location: [48.137154, 11.576124] as [number, number], // Munich center
			image: '/images/cars/green.jpg',
			details: { transmission: 'Manual', seats: 5, type: 'Compact', rating: 4.8 }
		},
		{
			id: 2,
			name: 'BMW 3 Series',
			price: 75,
			image: '/images/cars/green.jpg',
			location: [48.135124, 11.581234] as [number, number], // Slightly east
			details: { transmission: 'Automatic', seats: 5, type: 'Sedan', rating: 4.9 }
		},
		{
			id: 3,
			name: 'Audi A4',
			image: '/images/cars/green.jpg',
			price: 80,
			location: [48.132154, 11.579124] as [number, number], // Slightly south
			details: { transmission: 'Automatic', seats: 5, type: 'Sedan', rating: 4.7 }
		}
	];

	// Create circular buffer for infinite scroll
	let cars = [...carData, ...carData, ...carData];
	let selectedIndex = 0;
	let bottomSheetExpanded = spring(0);
	let carsSection: HTMLElement;
	let marker: HTMLElement;
	let isScrolling = false;
	let cardHeight = 0;
	let scrollTimeout: NodeJS.Timeout;

	$: sheetHeight = $bottomSheetExpanded * 400 + 80;
	$: selectedLocation = cars[selectedIndex]?.location || [48.137154, 11.576124];

	function updateSelectedCard() {
		if (!carsSection) return;

		const scrollTop = carsSection.scrollTop;
		cardHeight = cardHeight || carsSection.querySelector('.car-card')?.offsetHeight || 0;

		if (cardHeight) {
			selectedIndex = Math.round(scrollTop / cardHeight);
			if (!isScrolling) {
				const targetScroll = selectedIndex * cardHeight;
				if (scrollTop !== targetScroll) {
					carsSection.scrollTo({
						top: targetScroll,
						behavior: 'smooth'
					});
				}
			}
		}
	}

	function handleScroll() {
		isScrolling = true;
		updateSelectedCard();
		clearTimeout(scrollTimeout);
		scrollTimeout = setTimeout(() => {
			isScrolling = false;
			updateSelectedCard();
		}, 150);
	}

	function handleKeyDown(event: KeyboardEvent) {
		if (event.key === 'Enter' || event.key === ' ') {
			$bottomSheetExpanded = $bottomSheetExpanded ? 0 : 1;
		}
	}

	function toggleBottomSheet() {
		$bottomSheetExpanded = $bottomSheetExpanded ? 0 : 1;
	}

	onMount(() => {
		cardHeight = carsSection.querySelector('.car-card')?.offsetHeight || 0;
		updateSelectedCard();
	});
</script>

<div class="app-container">
	<!-- Map Section -->
	<div class="map-section">
		<Map {selectedLocation} zoom={13} />
	</div>

	<!-- Cars List Section -->
	<div
		class="cars-section"
		bind:this={carsSection}
		on:scroll={handleScroll}
		style="height: calc(100% - {sheetHeight}px)"
	>
		<div class="scroll-container">
			{#each cars as car, i (car.id + i)}
				<div class="car-card" class:selected={i === selectedIndex}>
					<div class="car-image-container">
						<img src={car.image} alt={car.name} class="car-image" loading="lazy" />
					</div>
					<div class="car-info">
						<div class="car-header">
							<h3>{car.name}</h3>
							<span class="price">€{car.price}/day</span>
						</div>
						<div class="car-details">
							<span>{car.details.type} • {car.details.transmission}</span>
							<span class="rating">★ {car.details.rating}</span>
						</div>
					</div>
				</div>
			{/each}
		</div>
	</div>

	<!-- Bottom Sheet -->
	<div class="bottom-sheet" style="height: {sheetHeight}px">
		<button class="bottom-sheet-header" on:click={toggleBottomSheet} on:keydown={handleKeyDown}>
			<div class="handle"></div>
			<h2>Search Options</h2>
		</button>

		<div class="bottom-sheet-content" style="opacity: {$bottomSheetExpanded}">
			<form class="search-options">
				<div class="search-group">
					<label for="location">Pick-up Location</label>
					<input id="location" type="text" placeholder="Munich, Germany" />
				</div>

				<div class="search-group dates">
					<div>
						<label for="pickup-date">Pick-up Date</label>
						<input id="pickup-date" type="date" />
					</div>
					<div>
						<label for="return-date">Return Date</label>
						<input id="return-date" type="date" />
					</div>
				</div>

				<div class="search-group">
					<label for="car-type">Car Type</label>
					<select id="car-type">
						<option>All Types</option>
						<option>Economy</option>
						<option>Luxury</option>
						<option>SUV</option>
					</select>
				</div>

				<button type="submit" class="search-button"> Search Cars </button>
			</form>
		</div>
	</div>
</div>

<style>
	.app-container {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		display: flex;
		flex-direction: column;
		background: #f5f5f5;
	}

	.map-section {
		height: 40vh;
		background: #e0e0e0;
		position: relative;
	}

	.mock-map {
		width: 100%;
		height: 100%;
		background: linear-gradient(135deg, #e0e0e0 0%, #f0f0f0 100%);
		position: relative;
	}

	.mock-marker {
		position: absolute;
		transform: translate(-50%, -50%);
		font-size: 24px;
		transition: all 0.3s ease-out;
	}

	.cars-section {
		flex-grow: 1;
		overflow-y: scroll;
		-webkit-overflow-scrolling: touch;
		margin-top: -20px;
		border-radius: 20px 20px 0 0;
		background: #f5f5f5;
		position: relative;
		z-index: 1;
		scrollbar-width: none;
		-ms-overflow-style: none;
	}

	.cars-section::-webkit-scrollbar {
		display: none;
	}

	.scroll-container {
		padding: 20px;
		padding-bottom: calc(100vh - 40px);
	}

	.car-card {
		background: white;
		border-radius: 16px;
		overflow: hidden;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
		transition: all 0.3s ease;
		margin-bottom: 16px;
		opacity: 0.8;
		transform: scale(0.98);
	}

	.car-card.selected {
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
		opacity: 1;
		transform: scale(1);
	}

	.car-image-container {
		width: 100%;
		height: 160px;
		overflow: hidden;
	}

	.car-image {
		width: 100%;
		height: 100%;
		background: linear-gradient(135deg, #e0e0e0 0%, #f0f0f0 100%);
	}

	.car-info {
		padding: 16px;
	}

	.car-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 8px;
	}

	.car-header h3 {
		font-size: 18px;
		font-weight: 600;
		margin: 0;
	}

	.price {
		font-size: 18px;
		font-weight: 600;
		color: #007aff;
	}

	.car-details {
		display: flex;
		justify-content: space-between;
		color: #666;
		font-size: 14px;
	}

	.rating {
		color: #ffb800;
	}

	.bottom-sheet {
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
		background: white;
		border-radius: 20px 20px 0 0;
		box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.1);
		transition: height 0.3s ease;
		z-index: 2;
	}

	.bottom-sheet-header {
		width: 100%;
		padding: 16px;
		text-align: center;
		border: none;
		background: none;
		border-bottom: 1px solid #eee;
		cursor: pointer;
	}

	.handle {
		width: 40px;
		height: 4px;
		background: #e0e0e0;
		border-radius: 2px;
		margin: 0 auto 16px;
	}

	.bottom-sheet-header h2 {
		margin: 0;
		font-size: 16px;
		font-weight: 600;
	}

	.bottom-sheet-content {
		padding: 20px;
		transition: opacity 0.3s ease;
	}

	.search-options {
		display: flex;
		flex-direction: column;
		gap: 20px;
	}

	.search-group {
		display: flex;
		flex-direction: column;
		gap: 8px;
	}

	.search-group.dates {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 16px;
	}

	label {
		font-size: 14px;
		font-weight: 500;
		color: #666;
	}

	input,
	select {
		padding: 12px;
		border: 1px solid #e0e0e0;
		border-radius: 10px;
		font-size: 16px;
		background: #f8f8f8;
	}

	input:focus,
	select:focus {
		outline: none;
		border-color: #007aff;
		background: white;
	}

	.search-button {
		background: #007aff;
		color: white;
		border: none;
		border-radius: 10px;
		padding: 14px;
		font-size: 16px;
		font-weight: 600;
		cursor: pointer;
		transition: background-color 0.2s ease;
	}

	.search-button:active {
		background: #0056b3;
	}
	.car-image {
		width: 100%;
		height: 100%;
		object-fit: cover;
		transition: transform 0.3s ease;
	}
</style>
