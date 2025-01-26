<script>
    import { Button } from "@sveltestrap/sveltestrap";
    import { get } from "svelte/store";

    const alfabet = "abcdefghijklmnopqrstuvwxyz";
    const alfabetArray = alfabet.split("");

    let letters = getWord();

    function getWord() {
        let randomLetter = alfabetArray[Math.floor(Math.random() * alfabetArray.length)];
        fetch(`https://api.datamuse.com/words?sp=${randomLetter}*`)
            .then((response) => response.json())
            .then((values) => {
                const randomWord = values[Math.floor(Math.random() * values.length)].word;
                letters = randomWord.toUpperCase().split('');
                console.log(letters);
            })
            .catch((error) => console.error(error));
    }

    function scanWord(){
        const inputs = document.querySelectorAll('input');
        let index = 0;
        inputs.forEach(input => {
            if(input.value.toUpperCase() === letters[index]){
                input.style.backgroundColor = 'green';
                input.readOnly = true;
            } else {
                input.style.backgroundColor = 'red';
            }
            index++;
        });
    }

</script>

<div class="d-flex justify-content-center align-items-center flex-column" style="height: 100vh;">
    <div class="outputText mb-5">
        {#each letters as letter}
            <input type="text" maxlength="1"/>
        {/each}
    </div>
    <button class="btn btn-secondary" on:click={scanWord}>Submit</button>
</div>

<style>
    .outputText {
        font-size: 4rem;
    }

    .outputText input {
        width: 5rem;
        border: none;
        outline: none;
        text-align: center;
        margin-right: 20px;
        display: inline-block;
        text-transform: uppercase;
        background-color: transparent;
        border-bottom: solid 2px rgb(20, 181, 230);
    }
</style>
