Hi this is Vihaan Gandhi
I am in 9th Standard and I like to code
I am good at cooking too
I live in Mumbai
This code is of snake game please enjoy and learn coding
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        #divScore{
            display:block;
            width:200px;
            margin:0 auto;
            font-size: 20px;
            text-align: center;
        }

        .gameTable{
            margin: 0 auto;
        }

        .blockOff{
            width: 20px;
            height: 20px;
            display: inline-block;
            border: 1px solid black;
            margin-right:1px;
            margin-bottom:1px;
        }

        .blockSnakeTail{
            width: 20px;
            height: 20px;
            display:inline-block;
            background-color: black;
            border: 1px solid black;
            margin-right:1px;
            margin-bottom:1px;
        }

        .blockSnakeHead{
            width: 20px;
            height: 20px;
            display: inline-block;
            background-color: green;
            border: 1px solid green;
            margin-right:1px;
            margin-bottom:1px;
        }

        .blockSnakeRodent{
            width: 20px;
            height: 20px;
            display: inline-block;
            background-color: red;
            border: 1px solid red;
            margin-right:1px;
            margin-bottom:1px;
        }
    </style>
</head>
<body>
    Enter size: <input id="txtSize" type="text"/> <br/>
    <input type="button" id="btnStart" value="Start"/><br/>

    </div>
    <script>
        let divGame = document.querySelector("#divGame");
        let divScore;
        let snake = {head: {x:0,y:0}, tail:[]};
        let rodent = {x:0,y:0};
        let direction;
        let size;
        let speed = 250;
        let lost = false;
        let score = 1;
        let frameInterval;
        document.querySelector('#btnStart').addEventListener('click', function(){
            size = document.querySelector('#txtSize').value;

            //Add the message labels
            divScore = document.createElement('div');
            divScore.setAttribute('id', 'divScore');
            document.body.appendChild(divScore);

            //construct the grid
            let table = document.createElement('table');
            table.setAttribute('cellspacing', 0);
            table.setAttribute('class', 'gameTable');
            document.body.appendChild(table);
            for(let i = 0;i<size;i++){
                let row = document.createElement('tr');
                table.appendChild(row);
                for(let j = 0;j<size;j++){
                    let block = document.createElement('td');
                    block.setAttribute('class', 'blockOff');
                    block.setAttribute('id', ['block', j, i].join(''));
                    row.appendChild(block);
                }
            }

            //create the snake
            //get random head x and y            
            let xVal, yVal;
            xVal = getRandomInt(0, size - 1);
            yVal = getRandomInt(0, size - 1);
            snake.head = {x:xVal,y:yVal};

            //create the tail. get a random direction, if valid put a tail on it.
            direction = getRandomInt(1, 4);
            let block = getDirectionalBlock(snake.head.x, snake.head.y, direction);
            while(!isblockValid(block.x, block.y, size, snake)){
                direction = getRandomInt(1, 4);
                block = getDirectionalBlock(snake.head.x, snake.head.y, direction);
            }

            snake.tail.push({x: block.x, y:block.y});

            //set the opposite direction for the movement
            if(direction === 1){ //left
                direction = 2;
            }
            else if(direction === 2){ //right
                direction = 1;
            }
            else if(direction === 3){ //top
                direction = 4;
            }
            else if(direction === 4){ //bottom
                direction = 3;
            }

            // --end of part 1

            // --begin part 2

            //get the random rodent
            rodent = getRandomRodent(size, snake);          

            //update initial score
            updateScore(score);

            //draw initial grid
            redrawGrid(size, table, snake, rodent);

            //start the frame interval
            setTimeout(() => {
                startFrames(size, table, snake, rodent);
            }, 2000);

            // --end of part 2

            // --begin part 3

            //key down events
            document.addEventListener('keydown', function(event) {
                let block;
                let currentDirection = direction;
                if(event.keyCode == 38) { //Up
                    direction = 3;                                
                }
                else if(event.keyCode == 39) { //Right
                    direction = 2;
                }
                else if(event.keyCode == 37) { //Left
                    direction = 1;
                }
                else if(event.keyCode == 40) { //Down
                    direction = 4;
                }

                block = getDirectionalBlock(snake.head.x, snake.head.y, direction);
                if(!isblockValid(block.x, block.y, size, snake)){
                    direction = currentDirection;
                }
            });

        });        

        function startFrames(size, table, snake, rodent){
            let intervalFunction = () => {
                //based on the direction, set the new snake and rodent position.
                let nextBlock = getDirectionalBlock(snake.head.x, snake.head.y, direction);

                //if the directional block is invalid then game over
                if(!isblockValid(nextBlock.x, nextBlock.y, size, snake)){
                    //Game over
                    gameLost();
                    clearInterval(frameInterval);
                }
                else {
                    //if the next block is rodent, then increase the snake tail and get new rodent
                    if(nextBlock.x === rodent.x && nextBlock.y === rodent.y){
                        score++;
                        speed -= 10;
                        if(speed < 10) speed = 10;

                        //clear interval and set with new speed
                        clearInterval(frameInterval);
                        frameInterval = setInterval(intervalFunction, speed);

                        updateScore(score);

                        //move snake head to tail array start
                        snake.tail = [{x: snake.head.x, y: snake.head.y}, ...snake.tail];

                        //set snake head as rodent position
                        snake.head = {x: rodent.x, y:rodent.y};

                        //get new rodent position
                        rodent = getRandomRodent(size, snake);
                    }
                    else {
                        //move the snake in the next direction
                        let tempHead = {x: snake.head.x, y:snake.head.y};
                        snake.head = {x: nextBlock.x, y:nextBlock.y};


                        //shift the snake tail blocks                        
                        for(let i=snake.tail.length - 1;i>=1;i--){
                            snake.tail[i] = {x: snake.tail[i-1].x, y: snake.tail[i-1].y};
                        }

                        snake.tail[0] = {x: tempHead.x, y:tempHead.y};
                    }

                    redrawGrid(size, table, snake, rodent);
                }


            }
            frameInterval = setInterval(intervalFunction, speed);
        }

        function redrawGrid(size, table, snake, rodent){
            for(let i=0;i<size;i++){
                for(let j=0;j<size;j++){
                    table.querySelector(['#block', i, j].join('')).setAttribute('class', 'blockOff');                    
                }
            }

            table.querySelector(['#block', snake.head.x, snake.head.y].join('')).setAttribute('class', 'blockSnakeHead');
            snake.tail.forEach(obj => {
                table.querySelector(['#block', obj.x, obj.y].join('')).setAttribute('class', 'blockSnakeTail');
            });

            table.querySelector(['#block', rodent.x, rodent.y].join('')).setAttribute('class', 'blockSnakeRodent');
        }

        function getRandomInt(min, max) {
            min = Math.ceil(min);
            max = Math.floor(max);
            return Math.floor(Math.random() * (max - min + 1)) + min;
        }

        function getRandomRodent(size, snake){
            let availableBlocks = getAvailableBlocks(size, snake);
            let randomIdx = getRandomInt(0, availableBlocks.length - 1);
            return availableBlocks[randomIdx];
        }

        function isblockValid(x, y, size, snake){
            let valid = true;

            //if block is out of bounds
            valid = (x >= 0 && x <= size - 1) && (y >= 0 && y <= size - 1);

            //if the block is on the snake
            if(valid && x === snake.head.x && y === snake.head.y){
                valid = false;
            }

            if(valid && snake.tail.length){
                for(let i=0;i<snake.tail.length;i++){
                    if(x === snake.tail[i].x && y === snake.tail[i].y){
                        valid = false; 
                        break;                       
                    }
                }
            }

            return valid;
        }

        function getDirectionalBlock(x, y, direction){
            //1 = left
            //2 = right
            //3 = top
            //4 = bottom

            switch(direction){
                case 1: //left
                    return {x : x - 1,y : y};
                case 2: //right
                    return {x : x + 1,y : y};
                case 3: //top
                    return {x : x,y : y - 1};
                case 4: //bottom
                    return {x : x,y : y + 1};
            }
        }

        function getAvailableBlocks(size, snake){
            let blocks = [];
            let snakeArr = [...snake.tail, snake.head];

            for(let i=0;i<size;i++){
                for(let j=0;j<size;j++){
                    let block = {x:i, y:j};
                    if(!arrayHasBlock(snakeArr, block)){
                        blocks.push(block);
                    }
                }
            }

            return blocks;
        }

        function arrayHasBlock(blockArray, block){
            blockArray.forEach(obj => {
                if(obj.x === block.x && obj.y === block.y){
                    return true;
                }
            });
            return false;
        }

        function updateScore(score){
            divScore.innerHTML = `Score: ${score}`;
        }

        function gameLost(){
            divScore.innerHTML = `${divScore.innerHTML}, YOU LOST`;
            lost = true;
        }
    </script>
</body>
</html>
