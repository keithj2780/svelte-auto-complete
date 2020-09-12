<script>
	import AutoComplete from './AutoComplete.svelte';
	import {cityList} from './data.js';
</script>

<AutoComplete items={cityList} 
	key="name"
	fromStart={false}
	placeholder='Your fav holiday spot'
	on:input={(e)=>{console.log('input '+e.detail);}}
	on:change={(e)=>{console.log('change', JSON.stringify(e.detail));}}
	 />
