<script>

	import Test from '../lib/Test.svelte';
	import Test2 from '../lib/Test2.svelte';
	import Test22 from '../lib/Test22.svelte';
	import Test3 from '../lib/Test3.svelte';
	import Test33 from '../lib/Test33.svelte';
	import TimelineV1 from '../lib/TimelineV1.svelte';
	import TimelineV1_mobile from '../lib/TimelineV1_mobile.svelte';

	export let data = {};

	$: artdat = data.data;
</script>

<!-- <Test {artdat}/> -->
<!-- <Test2/> -->
<!-- <Test3/> -->
<!-- <Test22 {artdat}/> -->
<!-- <Test33 {artdat}/> -->
<!-- <TimelineV1 {artdat}/> -->
<TimelineV1_mobile {artdat}/>