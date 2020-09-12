<svelte:window on:click={()=>close()} />

<script>
    import {createEventDispatcher } from 'svelte';
    const regExpEscape = (s) => {
        return s.replace(/[-\\^$*+?.()|[\]{}]/g, "\\$&");
    }

    //  Two elements - input & dropdown
    let input;
    let list;

    export let placeholder='Your location';
    export let required = false;
    export let disabled = false;

    // autocomplete props
    export let items= [];
    export let key;

    // options
    export let minChar= 2;
    export let maxItems= 10;
    export let fromStart= true;

    let isOpen= false;
    let results= [];
    let search= '';
    let arrowCounter= 0;

    let dispatch = createEventDispatcher();

function onChange (event) {
    dispatch('input', search)
    if (search.length >= Number(minChar)) {
        filterResults();
        isOpen = true;
    }
}

function filterResults () {
    let cnt = 0;
    // console.log(search,key,items.length);
    results = items.filter((item,idx) => {
        if (typeof item !== 'string') {
          item = item[key] || '';       // Silent fail
        }
        if (cnt >= maxItems) return false;
        //console.log(item);
        let match = fromStart ? item.toUpperCase().startsWith(search.toUpperCase())
                         : item.toUpperCase().includes(search.toUpperCase());
        if (match) cnt++;
        return match;
      });
    //   console.log('mapping '+results.length);
      results = results.map(item => {
        const text = typeof item !== 'string' ? item[key] : item
        return {
          key:      text,
          value:    item[key].value || item,
          label:    search.trim() === '' ? 
                        text :
                        text.replace(RegExp(regExpEscape(search.trim()), 'i'), "<span>$&</span>")
        };
      });
      //console.log(results);
      //const height = results.length > maxItems ? maxItems : results.length;
      const height = results.length;
      list.style.height = `${height * 2.05}rem`;
    //console.log('mapped '+results.length);
    if (arrowCounter >= results.length) arrowCounter = results.length-1;
}

function     onKeyDown (event) {
    //console.log('onKeyDown() '+event.keyCode+'  "'+search+'"'+'  arrowCounter='+arrowCounter);
    if (event.keyCode === 40) {    // ArrowDown
        if (arrowCounter === -1) arrowCounter++;
        else if (arrowCounter < results.length-1) arrowCounter++;
        event.preventDefault();
    } else if (event.keyCode === 38 && arrowCounter > 0) {  // ArrowUp
        arrowCounter--;
        event.preventDefault();
   } else if (event.keyCode === 13) {                      // Enter
        event.preventDefault();
        if (arrowCounter === -1) {
            arrowCounter = 0;            // Default select first item of list
        }
        close(arrowCounter);
    } else if (event.keyCode === 27) {      // Escape
        event.preventDefault();
        close();
    } else if (search.length+1 < Number(minChar)) {     //  when they backspace to < minChars, close the dropdown
        if (isOpen) {   isOpen= false;  arrowCounter=-1;    }
    }
}

function     close (index = -1) {
      isOpen= false;
      arrowCounter= -1;
      input.blur();             //  remove focus
      if (index > -1) {
        const { key, value } = results[index];
        dispatch('change', {value:value, key:key});
        search = key;
      }
    }
  
</script>

<div on:click|stopPropagation={()=>{}} class="autocomplete">
    <input
      type="text"
      {placeholder}
      {required}
      {disabled}
      bind:value={search}
      on:input={(e)=>onChange(e)}
      on:keydown={(e)=>onKeyDown(e)}
      bind:this={input}
    >
    <ul class="autocomplete-results{!isOpen ? ' hide-results' : ''}" bind:this={list}>
        {#each results as result, i}
            <li on:click={()=>close(i)}
                on:mousemove={()=>arrowCounter=i}
                class="autocomplete-result { i === arrowCounter ? ' is-active' : '' }">
                {@html result.label}
            </li>
        {/each}
    </ul>
  </div>
  

<style>
  * {
    box-sizing: border-box;
  }

  input {
    height: 2rem;
    font-size: 1rem;
    padding: 0.25rem 0.5rem;
  }

  .autocomplete {
    position: relative;
  }

  .hide-results {
    display: none;
  }

  .autocomplete-results {
    padding: 0;
    margin: 0;
    border: 1px solid #dbdbdb;
    height: 6rem;
    overflow: auto;
    width: 100%;

    background-color: white;
    box-shadow: 2px 2px 24px rgba(0, 0, 0, 0.1);
    position: absolute;
    z-index: 100;
  }

  .autocomplete-result {
    color: #7a7a7a;
    list-style: none;
    text-align: left;
    height: 2rem;
    padding: 0.25rem 0.5rem;
    cursor: pointer;
  }

  .autocomplete-result > :global(span) {
    background-color: none;
    color: #242424;
    font-weight: bold;
  }

  .autocomplete-result.is-active
  /* ,.autocomplete-result:hover  */
  {
    background-color: #dbdbdb;
  }
</style>