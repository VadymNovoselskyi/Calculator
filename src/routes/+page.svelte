<script>
    const OPERATORS_ORDER = ['/×', '+-'];
    let displayValue = '';

    function validateDidgit(event) {
        const digit = event.target.innerText;

        //does not allow '0' to be the beginning of some number (otherwise ex 01 would be possible)
        if(digit == 0 && !(/[\d.]$/.test(displayValue))) return;

        //add digit to the display and memory
        displayValue += digit;
    }

    function validateOperator(event) {
        const operator = event.target.innerText;
        //check if displayValue ends with a non-digit ('+', '-', '×' or '/')
        if (/\D$/.test(displayValue) || !(/\d$/.test(displayValue))) {
            if(operator === '-') displayValue += '(-)';
            return;
        }

        displayValue += operator;
    }

    function validateComma() {
        if (/[\+\-×\/]$/.test(displayValue)) {
            displayValue = '0.';
            return;
        }
        const regex = /(?<=\.)\d+$/;

        if(regex.test(displayValue) || /\.$/.test(displayValue)) return;

        displayValue += '.';
    }

    function calculate() {
        let answer = 0;
        const deletedIndexes = [];
        
        displayValue = displayValue.replace(/\D+$/, '');

        const values = (displayValue.split(/(?<!\(\-)[\+\-×\/](?!\))/g).map(number => number.replace(/\(-\)/g, '-')));
        const operators = (displayValue.replace(/\(-\)/g, '').match(/[\+\-×\/]/g));
        console.table(values);  
        console.table(operators);

        if(!values || !operators) {
            clear();
            return;
        }

        OPERATORS_ORDER.forEach(searchedOperator => {
            operators.forEach((operator, index) => {
                if (!searchedOperator.includes(operator)) return;
                
                index -= deletedIndexes.reduce((sum, deletedIndex) => sum += deletedIndex < index ? 1 : 0, 0);
                const result = handleArithmetics(operator, Number(values[index]), Number(values[index + 1])); 
                values[index] = result;
                values.splice(index + 1, 1);
                deletedIndexes.push(index);

                answer = result;
            });
        });
        clear();
        displayValue = `${answer}`;
    }

    function handleArithmetics(operator, number1, number2) {
        switch (operator) {
            case '/': return number1 / number2;
            case '×': return number1 * number2;
            case '+': return number1 + number2;
            case '-': return number1 - number2;
        }
    }

    function clear() {
        displayValue = '';
    }
</script>

<svelte:head>
    <title>Kalkylator</title>
</svelte:head>

<main>
    <input type="text" id = 'lcd' value={displayValue} disabled>
    <section id = 'keyBoard'>
        <button on:click={validateDidgit}>7</button>
        <button on:click={validateDidgit}>8</button>
        <button on:click={validateDidgit}>9</button>
        <button on:click={validateOperator}>+</button>

        <button on:click={validateDidgit}>4</button>
        <button on:click={validateDidgit}>5</button>
        <button on:click={validateDidgit}>6</button>
        <button on:click={validateOperator}>-</button>

        <button on:click={validateDidgit}>1</button>
        <button on:click={validateDidgit}>2</button>
        <button on:click={validateDidgit}>3</button>
        <button on:click={validateOperator}>×</button>
        
        <button on:click={validateComma}>,</button>
        <button on:click={validateDidgit}>0</button>
        <button on:click={calculate}>=</button>
        <button on:click={validateOperator}>/</button>

        <button id='clear' style = 'grid-column: span 4;'  on:click={clear}>CLEAR</button>
    </section>
</main>