<script>
    import { error } from "@sveltejs/kit";
    import { Button } from "@sveltestrap/sveltestrap";
    import { get } from "svelte/store";
    import { onMount } from "svelte";
    import HintButton from "./HintButton.svelte";

    const alfabet = "abcdefghijklmnopqrstuvwxyz";
    const alfabetArray = alfabet.split("");

    let letters = [];
    let definition = [];
    let inputValue = '';
    let submitBtn;
    let statusText;

    onMount(() => {
        getWord();
    });

    let score = 0;

    function getWord() {
       //get random word
       return fetch(`https://random-word-api.herokuapp.com/word?number=1`)
            .then((response) => {
                if(!response.ok){
                    throw new Error('Network response was not ok. Status: ' + response.status);
                }
                return response.json();
            })
            .then((values) => {
                const randomWord = values[0];
                letters = randomWord.toUpperCase().split('');
                getDefinition();
                //delete this
                console.log(letters);
            })
            .catch  (error => {console.error('Error: ', error)});
    }

    function getDefinition(){
        //get definition of the word
        fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${letters.join('')}`)
            .then(response => {
                if(!response.ok){
                    getWord();
                    throw new Error('Network response was not ok Status: ' + response.status);
                }
                return response.json();
            })
            .then(data => {
                definition = data[0].meanings[0].definitions;
                prepareDefinition();
                changeDefinition(definitionCount);
            })
            .catch(error => {console.error('Error: ', error)});
    }

    function prepareDefinition(){
        definitionCount = 0;
        for(let i = 0; i < definition.length; i++){
            definition[i] = definition[i].definition;
        }
    }

    //change the definition
    let definitionCount = 0;
    let definitionExport;
    
    function changeDefinition(num){
        if(num >= 0){
            if(definition[num] !== undefined){
                definitionExport = definition[num];
            } else {
                definitionCount = definition.length - 1;
            }
        } else {
            definitionCount = 0;
        }
    }
    $: changeDefinition(definitionCount);

    function scanWord(){
        const inputs = document.querySelectorAll('input');
        let index = 0;
        inputs.forEach(input => {
            //check if one input is correct
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
                statusText = `Correct! ${letters.join('')}`;
            }
            index++;
        });
    }

    function checkWord(e, index){
        if(statusText){
            return;
        }
        const input = e.target;
        if(input.value.toUpperCase() === letters[index]){
                input.style.backgroundColor = 'green';
                input.readOnly = true;
        } else {
                input.style.backgroundColor = 'red';
        }
        if (input.value === '') {
            input.focus();
            event.preventDefault();
        } else if (index >= letters.length - 1) {
            submitBtn.focus();
            scanWord();
        } else {
            const nextInput = document.querySelector(`input[data-index="${index + 1}"]`);
            nextInput.focus();
        }
    }

    //tohle moc nefunguje (potrebuju zmenit barvu kdyz se to smaze na transparent)
    function inputPress(e, index){
        if (e?.key === 'Backspace') {
            const prevInput = document.querySelector(`input[data-index="${index - 1}"]`);
            if (prevInput) {
                prevInput.focus();
            }
            e.target.value = '';
        }
    }

    function resetForm(){
        document.getElementById('form').reset();
        statusText = '';
        score++;
        getWord().then(() => {
            const inputs = document.querySelectorAll('input');
            inputs.forEach((input, index) => {
                input.readOnly = false;
                if(index === 0){
                    input.focus();
                }
                input.style.backgroundColor = 'transparent';
            });
        });
    }

    function onHintClick(){
        const randomIndex = Math.floor(Math.random() * letters.length);
        const inputs = document.querySelectorAll('input');
        inputs[randomIndex].value = letters[randomIndex];
    }
</script>

<div>
    <h1 class="text-center mt-5">Guess the word</h1>
    <div>Your score: {score}</div>
    <HintButton {onHintClick}/>
    {#if statusText}
        <div>{statusText}</div>
        <button class="btn btn-secondary" onclick={() => resetForm()}>Next word</button>
    {/if}
    <form id="form" class="d-flex justify-content-center align-items-center flex-column" style="height: 60vh;">
        <div class="justify-content-center align-items-center">
            <button class="btn btn-secondary" onclick={() => definitionCount--}>←</button>
            {definitionExport}
            <button class="btn btn-secondary" onclick={() => definitionCount++}>→</button>
        </div>
        <div class="outputText mb-5">
            {#each letters as letter, index (letter + index)}
                <input type="text" onkeydown={(e) => inputPress(e, index)} data-index={index} oninput={(e) => checkWord(e, index)} maxlength="1"/>
            {/each}
        </div>
        <button bind:this={submitBtn} class="btn btn-secondary" onclick={scanWord}>Submit</button>
    </form>
</div>

<style>
    .outputText {
        font-size: 4rem;
    }

    .outputText input {
        user-select: all;
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