var GAME = {
    width: 700,
    height: 700,
    background: new Image(),  // создаем объект для картинки бекграунда
    backgroundSrc: 'img/clock.png', // указываем путь до картинки
    totalLives: 3,
    currentLives: 3,
    score: 0,
    dafaultTimer: 5,
    currentTimer: 5,
    startTimerDate: 0,
    pointerSteps: 0,
    state: 'wait',
}
startTimer();

GAME.background.src = GAME.backgroundSrc;

function startTimer() {
    GAME.startTimerDate = new Date();
}

var CLOCK = {
    digitalMinutes: 11,
    digitalHours: 1,
    tickingMinutes: 39,
    tickingHours: 5,
    minutePointerLength: GAME.width / 3,
    hourPointerLength: GAME.width / 5,
}

var canvas = document.getElementById("canvas");
canvas.width = GAME.width;
canvas.height = GAME.height;
var canvasContext = canvas.getContext("2d");

function drawBackground() {
    canvasContext.fillStyle = GAME.background;
    canvasContext.drawImage(GAME.background, 0, 0, GAME.width, GAME.height);
}

function drawLives() {
    canvasContext.font = '48px serif';
    canvasContext.fillStyle = "black"
    canvasContext.fillText("Lives: " + GAME.currentLives, 10, 40);
}

function drawScore() {
    canvasContext.font = '48px serif';
    canvasContext.fillStyle = "black"
    canvasContext.fillText("Score: " + GAME.score, 10, 100);
}

function drawTimer() {
    canvasContext.font = '48px serif';
    canvasContext.fillStyle = "black"
    if (GAME.currentTimer < 3) {
        canvasContext.fillStyle = "red";
    }
    canvasContext.fillText("00:0" + GAME.currentTimer, 10, 160);
}

function drawDigitalClock() {
    canvasContext.font = '60px serif';
    canvasContext.fillStyle = "black"
    var hours = (CLOCK.digitalHours < 10)
        ? '0' + CLOCK.digitalHours
        : CLOCK.digitalHours
    var minutes = (CLOCK.digitalMinutes < 10)
        ? '0' + CLOCK.digitalMinutes
        : CLOCK.digitalMinutes
    canvasContext.fillText(hours + ":" + minutes, 400, 50);
}

function drawTempPointers() {
    if (GAME.state !== 'wait') {
        canvasContext.lineWidth = 2;
        canvasContext.lineCap = "round";
        canvasContext.beginPath();
        canvasContext.moveTo(GAME.width / 2, GAME.height / 2);

        let angle = GAME.pointerSteps * Math.PI / 30
        angle = angle - Math.PI / 2;

        let x = Math.cos(angle) * GAME.width / 2;
        let y = Math.sin(angle) * GAME.height / 2;

        canvasContext.lineTo(GAME.width / 2 + x, GAME.height / 2 + y);
        canvasContext.stroke();
    }
}

function drawMinutesPointer() {
    canvasContext.strokeStyle = 'black';
    if (false) {//проверить на текущее состояние game state
        canvasContext.strokeStyle = 'green';
    }
    canvasContext.lineWidth = 20;
    canvasContext.lineCap = "round";

    canvasContext.beginPath();
    canvasContext.moveTo(GAME.width / 2, GAME.height / 2);

    let angle = CLOCK.tickingMinutes * Math.PI / 30
    angle = angle - Math.PI / 2;

    let x = Math.cos(angle) * CLOCK.minutePointerLength;
    let y = Math.sin(angle) * CLOCK.minutePointerLength;

    canvasContext.lineTo(GAME.width / 2 + x, GAME.height / 2 + y);
    canvasContext.stroke();
}

function drawHoursPointer() {
    canvasContext.strokeStyle = 'black';
    if (false) {//проверить на текущее состояние game state
        canvasContext.strokeStyle = 'green';
    }
    canvasContext.lineWidth = 30;
    canvasContext.lineCap = "round";
    canvasContext.beginPath();
    canvasContext.moveTo(GAME.width / 2, GAME.height / 2);


    let angle = CLOCK.tickingHours * Math.PI / 30 * 5;
    angle = angle - Math.PI / 2;

    let x = Math.cos(angle) * CLOCK.hourPointerLength;
    let y = Math.sin(angle) * CLOCK.hourPointerLength;

    canvasContext.lineTo(GAME.width / 2 + x, GAME.height / 2 + y);
    canvasContext.stroke();
}

function drawFrame() {
    canvasContext.clearRect(0, 0, GAME.width, GAME.height);
    drawBackground();
    drawLives();
    drawScore();
    drawTimer();
    drawDigitalClock();
    drawTempPointers();
    drawHoursPointer();
    drawMinutesPointer();
}

function updateTimer() {
    if (GAME.state !== 'wait') {
        var diff = (new Date()).getTime() - GAME.startTimerDate.getTime();
        GAME.currentTimer = GAME.dafaultTimer - Math.floor(diff / 1000);

        if (GAME.currentTimer < 0) {
            GAME.currentTimer = 0
        }
    }
}

function initEventListeners() {
    window.addEventListener("mousemove", onCanvasMouseMove);
    window.addEventListener("mousemove", onCanvasMouseClick);
}

function onCanvasMouseMove(event) {
    let zeroVectorX = 0;
    let zeroVectorY = -GAME.width;
    let mouseVectorX = event.clientX - GAME.width / 2;
    let mouseVectorY = event.clientY - GAME.height / 2;

    let result = (zeroVectorX * mouseVectorX + zeroVectorY * mouseVectorY)
        / (Math.sqrt(zeroVectorX * zeroVectorX + zeroVectorY * zeroVectorY) * (Math.sqrt(mouseVectorX * mouseVectorX + mouseVectorY * mouseVectorY)))

    let resultAngle = Math.acos(result)

    if (event.clientX < GAME.width / 2) {
        resultAngle = 2 * Math.PI - resultAngle;
    }

    let steps = Math.round(resultAngle * 60 / (Math.PI * 2)) % 60;
    GAME.pointerSteps = steps;
}

function onCanvasMouseClick() {
    if (GAME.state === 'wait') {
        startTimer();
        GAME.state = 'hour';
        return;
    }

    if (GAME.state === 'hour') {
        CLOCK.tickingHours = GAME.pointerSteps / 5;
        GAME.state = 'minutes';
        return;
    }

    if (GAME.state === 'minutes') {
        CLOCK.ticking = GAME.pointerSteps / 5;
        checkResults();
        return;
    }
    
    // делать по клику, устанавливать значение поинтерСтепс в значение стрелок часов

}

function checkResults() {
    if (GAME.currentTimer <= 0) {
        GAME.currentLives = GAME.currentLives - 1;
        GAME.state = 'wait';
        return
    }
}
if (CLOCK.digitalHours = CLOCK.tickingHours) {
    GAME.score = GAME.score + 1;
    GAME.state = 'wait';
}

function randomInteger(min, max) {
    // случайное число от min до (max+1)
    let rand = min + Math.random() * (max + 1);
    return Math.floor(rand);
}

function drawVictory() {
    canvasContext.font = '64px Arial';
    canvasContext.fillStyle = "black";
    canvasContext.textAlign = 'center';
    canvasContext.fillText("VICTORY", GAME.width / 2, GAME.height / 2);
}

function drawFailed() {
    canvasContext.font = '64px Arial';
    canvasContext.fillStyle = "red";
    canvasContext.textAlign = 'center';
    canvasContext.fillText("FAILED", GAME.width / 2, GAME.height / 2);
}

function play() {
    if (GAME.score < 5) {
        drawFrame();
        updateTimer();
        requestAnimationFrame(play);
    }
    if (GAME.score >= 5) {
        drawBackground();
        drawVictory();
    }
    if (GAME.currentLives <= 0) {
        drawBackground();
        drawFailed();
    }
}

initEventListeners();
play();
