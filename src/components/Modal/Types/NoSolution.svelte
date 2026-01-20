<script>
    export let data ;
    export let hideModal;
    import { invalidCells } from '@sudoku/stores/grid';
    import { cursor } from "@sudoku/stores/cursor";
    function handleContinue() {
        if(data.callback)data.callback();

        hideModal();

    }
</script>

<h1 class="text-3xl font-semibold mb-5 leading-none">No Solution</h1>

<p class="text-lg mb-5">No Solution in current try, Go back find other way.</p>

<div class="flex justify-end">
    <button class="btn btn-small mr-3" on:click={hideModal}>Cancel</button>
    <button class="btn btn-small btn-primary" on:click={handleContinue}
        >Continue</button
    >
</div>
