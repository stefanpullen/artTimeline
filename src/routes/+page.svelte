<script>
	import TimelineV1 from '../lib/TimelineV1.svelte';
	import TimelineV1_mobile from '../lib/TimelineV1_mobile.svelte';
	import DeviceDetector from 'svelte-device-detector';

			export let data = {};
			$: artdat = data.data;
</script>



<DeviceDetector showInDevice="desktop">
	<TimelineV1 {artdat} />
</DeviceDetector>

<DeviceDetector showInDevice="mobile">
	<TimelineV1_mobile {artdat} />
</DeviceDetector>
