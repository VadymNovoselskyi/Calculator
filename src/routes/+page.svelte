<script>
    let displayValue = '';
    let values = [];
    let operators = [];

    function addDidgit(event) {
        const digit = event.target.innerText;
        const lastIndex = /[+\-x\/]$/.test(displayValue) //tests if the last char is '+', '-', 'x' or '/'
        ? values.length //if true, then new index at values shall be created
        : (values.length || 1) - 1; //if false, then digit shall be added to existing value at the last place

        //does not allow '0' to be the beginning of some number (otherwise ex 01 would be possible)
        if(digit == 0 && !(/\d$/.test(displayValue))) return;

        //add digit to the display and memory
        displayValue += digit;

        //values[lastIndex] ?? '' is needed when trying to add new index
        values[lastIndex] = (values[lastIndex] ?? '') + digit; 
    }

    function handleOperator(event) {
        const operator = event.target.innerText;
        //check if displayValue ends with a non-digit ('+', '-', 'x' or '/')
        if (/\D$/.test(displayValue)) {
            if(operator === '-') {
                values[0] = '-';
                displayValue += '(-)';
            }
            return;
        }

        operators.push(operator);
        displayValue += operator;
    }

    function addComma() {
        const lastIndex = (values.length || 1) - 1;
        if(values[lastIndex] === '' || !values[lastIndex]) {
            values[lastIndex] = '0.';
            displayValue += '0.';
            return;
        }
        else if(values[lastIndex].includes('.')) return;

        values[lastIndex] += '.';
        displayValue += '.';
    }

    function handleCalculation() {
        const operatorOrder = ['/x', '+-'];
        let answer = 0;
        const deletedIndexes = [];
        if (operators.length + 1 === values.length && !values[values.length - 1]) operators.pop();
        console.table(values)
        console.table(operators)

        operatorOrder.forEach(searchedOperator => {
            operators.forEach((operator, index) => {
                if (!searchedOperator.includes(operator)) return;
                
                index -= deletedIndexes.reduce((sum, deletedIndex) => sum += deletedIndex < index ? 1 : 0, 0);
                const result = handleMathOperation(operator, Number(values[index]), Number(values[index + 1])); 
                values[index] = result;
                values.splice(index + 1, 1);
                deletedIndexes.push(index);

                answer = result
            });
        });
        clear();
        displayValue = `${answer}`;
        values.push(answer);
    }

    function handleMathOperation(operator, number1, number2) {
        switch (operator) {
            case '/': return number1 / number2;
            case 'x': return number1 * number2;
            case '+': return number1 + number2;
            case '-': return number1 - number2;
        }
    }

    function clear() {
        displayValue = '';
        values = [];
        operators = [];
    }
</script>

<svelte:head>
    <title>Kalkylator</title>
</svelte:head>

<main>
    <input type="text" id = 'lcd' value={displayValue} disabled>
    <section id = 'keyBoard'>
        <button id='b7' on:click={addDidgit}>7</button>
        <button id='b8' on:click={addDidgit}>8</button>
        <button id='b9' on:click={addDidgit}>9</button>
        <button id='add' on:click={handleOperator}>+</button>

        <button id='b4' on:click={addDidgit}>4</button>
        <button id='b5' on:click={addDidgit}>5</button>
        <button id='b6' on:click={addDidgit}>6</button>
        <button id='sub' on:click={handleOperator}>-</button>

        <button id='b1' on:click={addDidgit}>1</button>
        <button id='b2' on:click={addDidgit}>2</button>
        <button id='b3' on:click={addDidgit}>3</button>
        <button id='mul' on:click={handleOperator}>x</button>
        
        <button id='comma' on:click={addComma}>,</button>
        <button id='b0' on:click={addDidgit}>0</button>
        <button id='enter' on:click={handleCalculation}>=</button>
        <button id='div' on:click={handleOperator}>/</button>

        <button id='clear' style = 'grid-column: span 4;'  on:click={clear}>CLEAR</button>
    </section>
</main>