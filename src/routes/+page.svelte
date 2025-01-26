<script>
    import { Button } from "@sveltestrap/sveltestrap";
    import { get } from "svelte/store";

    const alfabet = "abcdefghijklmnopqrstuvwxyz";
    const alfabetArray = alfabet.split("");

    let letters = getWord();
    let definition;

    function getWord() {
        //get random word
        let randomLetter = alfabetArray[Math.floor(Math.random() * alfabetArray.length)];
        fetch(`https://api.datamuse.com/words?sp=${randomLetter}*`)
            .then((response) => response.json())
            .then((values) => {
                const randomWord = values[Math.floor(Math.random() * values.length)].word;
                letters = randomWord.toUpperCase().split('');
                //delete this
                console.log(letters);
                //get definition of the word
                fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${letters.join('')}`)
                    .then((response) => response.json())
                    .then((data) => {
                        definition = data[0].meanings[0].definitions[0].definition;
                    })
                    .catch((error) => console.error(error));
            })
            .catch((error) => console.error(error));
    }

    function scanWord(){
        const inputs = document.querySelectorAll('input');
        let index = 0;
        //check if one input is correct
        inputs.forEach(input => {
            if(input.value.toUpperCase() === letters[index]){
                input.style.backgroundColor = 'green';
                input.readOnly = true;
            } else {
                input.style.backgroundColor = 'red';
            }

            //check if all inputs are correct
            const allReadOnly = Array.prototype.every.call(inputs, (input) => {
                return input.readOnly;
            });
            if(allReadOnly){
                resetForm();
                getWord();
                alert('You won! Correct word is: ' + letters.join(''));
            }
            index++;
        });
    }

    function resetForm(){
        document.getElementById('form').reset();
        const inputs = document.querySelectorAll('input');
        inputs.forEach(input => {
            input.readOnly = false;
            input.style.backgroundColor = 'transparent';
        });
    }
</script>

<div>
    <form id="form" class="d-flex justify-content-center align-items-center flex-column" style="height: 100vh;">
        <div>{definition}</div>
        <div class="outputText mb-5">
            {#each letters as letter}
                <input type="text" maxlength="1"/>
            {/each}
        </div>
        <button id="resetForm" class="btn btn-secondary" on:click={scanWord}>Submit</button>
    </form>
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
