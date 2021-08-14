<script>
import { Jumper } from 'svelte-loading-spinners'
</script>

<div>

</div>
<main>
<Jumper size="60" color="#FF3E00" unit="px" duration="1s"></Jumper>
</main>
<style>
    div {
        background-color: white;
        position: absolute;
        z-index: 1;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }

    main {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        z-index: 2;
    }    

</style>