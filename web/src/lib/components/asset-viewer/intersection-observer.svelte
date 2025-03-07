<script lang="ts">
	import { onMount } from 'svelte';
	import { createEventDispatcher } from 'svelte';

	export let once = false;
	export let top = 0;
	export let bottom = 0;
	export let left = 0;
	export let right = 0;
	export let root: HTMLElement | null = null;

	let intersecting = false;
	let container: any;
	const dispatch = createEventDispatcher();

	onMount(() => {
		if (typeof IntersectionObserver !== 'undefined') {
			const rootMargin = `${top}px ${right}px ${bottom}px ${left}px`;
			const observer = new IntersectionObserver(
				(entries) => {
					intersecting = entries[0].isIntersecting;
					if (!intersecting) {
						dispatch('hidden', container);
					}

					if (intersecting && once) {
						observer.unobserve(container);
					}

					if (intersecting) {
						dispatch('intersected', container);
					}
				},
				{
					rootMargin,
					root
				}
			);

			observer.observe(container);
			return () => observer.unobserve(container);
		}

		// The following is a fallback for older browsers
		function handler() {
			const bcr = container.getBoundingClientRect();

			intersecting =
				bcr.bottom + bottom > 0 &&
				bcr.right + right > 0 &&
				bcr.top - top < window.innerHeight &&
				bcr.left - left < window.innerWidth;

			if (intersecting && once) {
				window.removeEventListener('scroll', handler);
			}
		}

		window.addEventListener('scroll', handler);
		return () => window.removeEventListener('scroll', handler);
	});
</script>

<div bind:this={container}>
	<slot {intersecting} />
</div>
