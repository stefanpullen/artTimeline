<script>
	import TimelineV1_mobile from '../lib/TimelineV1_mobile.svelte';

	export let data = {};

	$: artdat = data.data;
</script>

<TimelineV1_mobile {artdat} />
