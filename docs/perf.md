```blocks
let levelTime: number
let person: Entity
let monsters: Entity[]
let totalMonsters: number
let playing: boolean
let gameSuspended: boolean
let busyPos: Point[]

class Entity {
    public x: number
    public y: number
    public dirX: number
    public dirY: number
    public hitHorizontalWall(): boolean {
        return this.y == 0 && this.dirY == -1 || this.y == 4 && this.dirY == 1
    }

    public hitVerticalWall(): boolean {
        return this.x == 0 && this.dirX == -1 || this.x == 4 && this.dirX == 1
    }

    public possHorizontalDir(): number {
        if (this.x == 0) {
            return 1
        } else if (this.x == 4) {
            return - 1
        } else {
            return Math.random(2) * 2 - 1
        }
    }

    public possVerticalDir(): number {
        if (this.y == 0) {
            return 1
        } else if (this.y == 4) {
            return - 1
        } else {
            return Math.random(2) * 2 - 1
        }
    }

    public collidesX(p2: Entity): boolean {
        return this.y == p2.y && this.y + this.dirY == p2.y + p2.dirY && (this.x + this.dirX == p2.x || this.x + this.dirX == p2.x + p2.dirX || p2.x + p2.dirX == this.x)
    }

    public collidesY(p2: Entity): boolean {
        return this.x == p2.x && this.x + this.dirX == p2.x + p2.dirX && (this.y + this.dirY == p2.y || this.y + this.dirY == p2.y + p2.dirY || p2.y + p2.dirY == this.y)
    }

    public move1() {
        this.x = this.x + this.dirX
        this.y = this.y + this.dirY
    }

    public towardsX(p2: Entity): number {
        return Math.sign(p2.x - this.x)
    }

    public towardsY(p2: Entity): number {
        return Math.sign(p2.y - this.y)
    }

    public plot() {
        led.plot(this.x, this.y)
    }

    public blink() {
        led.plot(this.x, this.y)
        basic.pause(125)
        led.unplot(this.x, this.y)
        basic.pause(125)
        led.plot(this.x, this.y)
    }

}

class Point {
    public x: number
    public y: number
}

initializeState()
redraw()
basic.pause(1000)
basic.forever(() => {
    levelTime = levelTime + 12
    basic.pause(12)
    if (!playing) {
        levelTime = 0
        playing = true
    }
    if (levelTime >= 5000) {
        gameSuspended = true
        game.levelUp()
        levelUp()
        levelTime = 0
        resetState()
        redraw()
        basic.pause(1000)
        gameSuspended = false
    }
})
basic.forever(() => {
    if (!gameSuspended) {
        logic()
        redraw()
        basic.pause(500)
    }
})
input.onButtonPressed(Button.A, () => {
    let temp = Math.abs(person.dirX) * (-1)
    person.dirX = Math.abs(person.dirY) * (-1)
    person.dirY = temp
})
input.onButtonPressed(Button.B, () => {
    let temp1 = Math.abs(person.dirX)
    person.dirX = Math.abs(person.dirY)
    person.dirY = temp1
})

function redraw() {
    basic.clearScreen()
    person.plot()
    for (let i = 0; i < totalMonsters; i++) {
        monsters[i].blink()
    }
}

function initializeState() {
    person = new Entity()
    playing = false
    busyPos = ([] as Point[])
    let busyPos1 = new Point()
    busyPos1.x = 1
    busyPos1.y = 1
    let busyPos2 = new Point()
    busyPos2.x = 1
    busyPos2.y = 3
    let busyPos3 = new Point()
    busyPos3.x = 3
    busyPos3.y = 1
    busyPos.push(busyPos1)
    busyPos.push(busyPos2)
    busyPos.push(busyPos3)
    monsters = ([] as Entity[])
    addMonster()
    resetState()
}

function logic() {
    if (person.hitHorizontalWall()) {
        person.dirY = 0
        person.dirX = person.possHorizontalDir()
    }
    if (person.hitVerticalWall()) {
        person.dirX = 0
        person.dirY = person.possVerticalDir()
    }
    let lost = false
    for (let i = 0; i < totalMonsters; i++) {
        let m = monsters[i]
        m.dirX = m.towardsX(person)
        m.dirY = m.towardsY(person)
        if (m.dirX != 0 && m.dirY != 0) {
            let x = Math.random(2)
            if (x == 1) {
                m.dirX = 0
            } else {
                m.dirY = 0
            }
        }
        if (person.collidesX(m) || person.collidesY(m)) {
            lost = true
        }
    }
    if (!lost) {
        moveAll()
    } else {
        loseLife()
    }
}

function loseLife() {
    moveAll()
    basic.pause(500)
    basic.showLeds(`
        . # . # .
        . . # . .
        . . . . .
        . # # # .
        # . . . #
        `, 400)
    basic.pause(1000)
    basic.clearScreen()
    game.removeLife(1)
    playing = false
    resetState()
}

function moveAll() {
    person.move1()
    for (let i = 0; i < totalMonsters; i++) {
        monsters[i].move1()
    }
}

function addMonster() {
    let m = new Entity()
    monsters.push(m)
    totalMonsters = totalMonsters + 1
}

function levelUp() {
    addMonster()
}

function resetState() {
    levelTime = 0
    game.setLife(5)
    person.x = 4
    person.y = 4
    person.dirX = -1
    person.dirY = 0
    for (let i = 0; i < totalMonsters; i++) {
        let busy = busyPos[i]
        let m = monsters[i]
        m.x = (busy.x + Math.random(3)) - 1
        m.y = (busy.y + Math.random(3)) - 1
    }
}

```

```blocks
let correctBall: number
let ballRevealing: boolean
let cupSelect: string
let index: number
let score: number
let level: number
let swapSpeed: number

initializeGame()
input.onButtonPressed(Button.A, () => {
    if (ballRevealing) {
        index = index + 1
        if (index > 2) {
            index = 0
        }
        basic.showString(cupSelect[index], 150)
    }
})
input.onButtonPressed(Button.B, () => {
    if (ballRevealing) {
        ballRevealing = false
        if (correctBall == index) {
            score = score + level
            images.createImage(`
                . . . . .
                . . . . #
                . . . # .
                # . # . .
                . # . . .
                `).showImage(0)
            basic.pause(1000)
            basic.showString("+".concat(level.toString()), 150)
            basic.pause(1000)
        } else {
            images.createImage(`
                # . . . #
                . # . # .
                . . # . .
                . # . # .
                # . . . #
                `).showImage(0)
            basic.pause(1000)
            basic.clearScreen()
            revealBall(correctBall)
            basic.pause(1000)
        }
    }
    level = level + 1
    if (level == 4) {
        basic.showString("FINAL SCORE:", 75)
        basic.showNumber(score, 150)
    } else {
        playLevel(level)
    }
})
playLevel(1)

function revealBall(p: number) {
    let xCoordinate = 2 * p
    for (let j = 0; j < 3; j++) {
        led.plot(j * 2, 2)
    }
    for (let i = 0; i < 3; i++) {
        led.unplot(xCoordinate, 2)
        led.plot(xCoordinate, 1)
        basic.pause(100)
        led.unplot(xCoordinate, 1)
        led.plot(xCoordinate, 0)
        basic.pause(200)
        led.unplot(xCoordinate, 0)
        led.plot(xCoordinate, 1)
        basic.pause(100)
        led.unplot(xCoordinate, 1)
        led.plot(xCoordinate, 2)
        basic.pause(75)
    }
    basic.pause(1000)
}

function initializeGame() {
    ballRevealing = false
    level = 1
    score = 0
    cupSelect = "LMR"
}

function swapCups(cup_1: number, cup_2: number, pauseDifficulty: number) {
    let cup_1X = 2 * cup_1
    let cup_2X = 2 * cup_2
    let cupXAverage = (cup_1X + cup_2X) / 2
    led.unplot(cup_1X, 2)
    led.unplot(cup_2X, 2)
    led.plot(cup_1X, 3)
    led.plot(cup_2X, 1)
    basic.pause(pauseDifficulty)
    led.unplot(cup_1X, 3)
    led.unplot(cup_2X, 1)
    led.plot(cup_1X, 4)
    led.plot(cup_2X, 0)
    basic.pause(pauseDifficulty)
    led.unplot(cup_1X, 4)
    led.unplot(cup_2X, 0)
    if (cupXAverage == 2) {
        led.plot((cupXAverage + cup_1X) / 2, 4)
        led.plot((cupXAverage + cup_2X) / 2, 0)
        basic.pause(pauseDifficulty)
        led.unplot((cupXAverage + cup_1X) / 2, 4)
        led.unplot((cupXAverage + cup_2X) / 2, 0)
    }
    led.plot(cupXAverage, 4)
    led.plot(cupXAverage, 0)
    basic.pause(pauseDifficulty)
    led.unplot(cupXAverage, 4)
    led.unplot(cupXAverage, 0)
    if (cupXAverage == 2) {
        led.plot((cupXAverage + cup_2X) / 2, 4)
        led.plot((cupXAverage + cup_1X) / 2, 0)
        basic.pause(pauseDifficulty)
        led.unplot((cupXAverage + cup_2X) / 2, 4)
        led.unplot((cupXAverage + cup_1X) / 2, 0)
    }
    led.plot(cup_2X, 4)
    led.plot(cup_1X, 0)
    basic.pause(pauseDifficulty)
    led.unplot(cup_2X, 4)
    led.unplot(cup_1X, 0)
    led.plot(cup_2X, 3)
    led.plot(cup_1X, 1)
    basic.pause(pauseDifficulty)
    led.unplot(cup_2X, 3)
    led.unplot(cup_1X, 1)
    led.plot(cup_2X, 2)
    led.plot(cup_1X, 2)
    basic.pause(pauseDifficulty)
    if (correctBall == cup_1) {
        correctBall = cup_2
    } else if (correctBall == cup_2) {
        correctBall = cup_1
    }
}

function swapFake(cup_1: number, cup_2: number, pauseDifficulty: number) {
    let cup_1X = 2 * cup_1
    let cup_2X = 2 * cup_2
    let cupXAverage = (cup_1X + cup_2X) / 2
    led.unplot(cup_1X, 2)
    led.unplot(cup_2X, 2)
    led.plot(cup_1X, 3)
    led.plot(cup_2X, 1)
    basic.pause(pauseDifficulty)
    led.unplot(cup_1X, 3)
    led.unplot(cup_2X, 1)
    led.plot(cup_1X, 4)
    led.plot(cup_2X, 0)
    basic.pause(pauseDifficulty)
    led.unplot(cup_1X, 4)
    led.unplot(cup_2X, 0)
    if (cupXAverage == 2) {
        led.plot((cupXAverage + cup_1X) / 2, 4)
        led.plot((cupXAverage + cup_2X) / 2, 0)
        basic.pause(pauseDifficulty)
        led.unplot((cupXAverage + cup_1X) / 2, 4)
        led.unplot((cupXAverage + cup_2X) / 2, 0)
    }
    led.plot(cupXAverage, 4)
    led.plot(cupXAverage, 0)
    basic.pause(pauseDifficulty)
    led.unplot(cupXAverage, 4)
    led.unplot(cupXAverage, 0)
    if (cupXAverage == 2) {
        led.plot((cupXAverage + cup_1X) / 2, 4)
        led.plot((cupXAverage + cup_2X) / 2, 0)
        basic.pause(pauseDifficulty)
        led.unplot((cupXAverage + cup_1X) / 2, 4)
        led.unplot((cupXAverage + cup_2X) / 2, 0)
    }
    led.plot(cup_1X, 4)
    led.plot(cup_2X, 0)
    basic.pause(pauseDifficulty)
    led.unplot(cup_1X, 4)
    led.unplot(cup_2X, 0)
    led.plot(cup_1X, 3)
    led.plot(cup_2X, 1)
    basic.pause(pauseDifficulty)
    led.unplot(cup_1X, 3)
    led.unplot(cup_2X, 1)
    led.plot(cup_1X, 2)
    led.plot(cup_2X, 2)
    basic.pause(pauseDifficulty)
}

function playLevel(level1: number) {
    basic.showNumber(level, 150)
    basic.pause(3000)
    basic.clearScreen()
    for (let i = 0; i < 3; i++) {
        led.plot(2 * i, 2)
    }
    basic.pause(1000)
    correctBall = Math.random(3)
    revealBall(correctBall)
    basic.pause(1000)
    let swaps = 5 + 10 * level1
    if (level1 == 1) {
        swapSpeed = 80
    } else if (level1 == 2) {
        swapSpeed = 40
    } else {
        swapSpeed = 20
    }
    for (let i1 = 0; i1 < swaps; i1++) {
        let swapType = Math.random(3)
        let not = Math.random(3)
        if (swapType < 2) {
            let swapOrientation = Math.random(2)
            if (swapOrientation == 0) {
                swapCups((not + 1) % 3, (not + 2) % 3, swapSpeed)
            } else {
                swapCups((not + 2) % 3, (not + 1) % 3, swapSpeed)
            }
        } else {
            let swapOrientation1 = Math.random(2)
            if (swapOrientation1 == 0) {
                swapFake((not + 1) % 3, (not + 2) % 3, swapSpeed)
            } else {
                swapFake((not + 2) % 3, (not + 1) % 3, swapSpeed)
            }
        }
    }
    index = -1
    ballRevealing = true
}
```

```blocks
let oneX: number
let oneY: number
let twoX: number
let twoY: number
let pause: number
let meteoriteOneX: number
let meteoriteOneY: number
let meteoriteTwoX: number
let meteoriteTwoY: number
let counter: number

basic.pause(2000)
oneX = 0
oneY = 4
twoX = 1
twoY = 4
counter = 0
pause = 700
led.plot(oneX, oneY)
led.plot(twoX, twoY)
input.onButtonPressed(Button.A, () => {
    if (oneX > 0) {
        led.unplot(oneX, oneY)
        led.unplot(twoX, twoY)
        oneX = oneX - 1
        twoX = twoX - 1
        led.plot(oneX, oneY)
        led.plot(twoX, twoY)
    }
})
input.onButtonPressed(Button.B, () => {
    if (twoX < 4) {
        led.unplot(oneX, oneY)
        led.unplot(twoX, twoY)
        oneX = oneX + 1
        twoX = twoX + 1
        led.plot(oneX, oneY)
        led.plot(twoX, twoY)
    }
})
meteoriteOneX = Math.random(5)
meteoriteOneY = 0
meteoriteTwoX = Math.random(5)
meteoriteTwoY = -3
basic.pause(1000)
for (let i = 0; i < 3; i++) {
    led.plot(meteoriteTwoX, meteoriteTwoY)
    led.plot(meteoriteOneX, meteoriteOneY)
    basic.pause(pause)
    led.unplot(meteoriteTwoX, meteoriteTwoY)
    led.unplot(meteoriteOneX, meteoriteOneY)
    meteoriteOneY = meteoriteOneY + 1
    meteoriteTwoY = meteoriteTwoY + 1
}
basic.forever(() => {
    for (let i1 = 0; i1 < 3; i1++) {
        led.plot(meteoriteTwoX, meteoriteTwoY)
        led.plot(meteoriteOneX, meteoriteOneY)
        basic.pause(pause)
        led.unplot(meteoriteOneX, meteoriteOneY)
        led.unplot(meteoriteTwoX, meteoriteTwoY)
        meteoriteOneY = meteoriteOneY + 1
        meteoriteTwoY = meteoriteTwoY + 1
        if (meteoriteOneY == 4) {
            if (meteoriteOneX == oneX) {
                for (let j = 0; j < 10; j++) {
                    led.plotAll()
                    basic.pause(200)
                    basic.clearScreen()
                    basic.pause(200)
                }
                basic.showNumber(counter, 150)
                basic.pause(10000)
            } else if (meteoriteOneX == twoX) {
                for (let j1 = 0; j1 < 10; j1++) {
                    led.plotAll()
                    basic.pause(200)
                    basic.clearScreen()
                    basic.pause(200)
                }
                basic.showNumber(counter, 150)
                basic.pause(10000)
            }
        }
    }
    while (Math.abs(meteoriteTwoX - meteoriteOneX) < 1) {
        meteoriteOneX = Math.random(5)
    }
    meteoriteOneY = 0
    counter = counter + 1
    if (counter == 3) {
        pause = pause - 250
    } else if (counter == 8) {
        pause = pause - 100
    } else if (counter == 12) {
        pause = pause - 100
    } else if (counter == 20) {
        pause = pause - 100
    } else if (counter == 30) {
        pause = pause - 70
    }
    if (counter == 40) {
        pause = pause - 70
    }
    for (let i2 = 0; i2 < 3; i2++) {
        led.plot(meteoriteOneX, meteoriteOneY)
        led.plot(meteoriteTwoX, meteoriteTwoY)
        basic.pause(pause)
        led.unplot(meteoriteOneX, meteoriteOneY)
        led.unplot(meteoriteTwoX, meteoriteTwoY)
        meteoriteOneY = meteoriteOneY + 1
        meteoriteTwoY = meteoriteTwoY + 1
        if (meteoriteTwoY == 4) {
            if (meteoriteTwoX == oneX) {
                for (let j2 = 0; j2 < 10; j2++) {
                    led.plotAll()
                    basic.pause(200)
                    basic.clearScreen()
                    basic.pause(200)
                }
                basic.showNumber(counter, 150)
                basic.pause(10000)
            } else if (meteoriteTwoX == twoX) {
                for (let j3 = 0; j3 < 10; j3++) {
                    led.plotAll()
                    basic.pause(200)
                    basic.clearScreen()
                    basic.pause(200)
                }
                basic.showNumber(counter, 150)
                basic.pause(10000)
            }
        }
    }

    meteoriteTwoX = Math.random(5)
    while (Math.abs(meteoriteTwoX - meteoriteOneX) < 1) {
        meteoriteTwoX = Math.random(5)
    }
    meteoriteTwoY = 0
    counter = counter + 1
    if (counter == 3) {
        pause = pause - 250
    } else if (counter == 8) {
        pause = pause - 100
    } else if (counter == 12) {
        pause = pause - 100
    } else if (counter == 20) {
        pause = pause - 100
    } else if (counter == 30) {
        pause = pause - 70
    } else if (counter == 40) {
        pause = pause - 70
    }
})

```

```blocks
let AWasPressed: boolean
let BWasPressed: boolean
let ABWasPressed: boolean
let wasShake: boolean
let scoreA: number
let scoreB: number

scoreA = 0
scoreB = 0
startIOMonitor()
let gameTime = getGameTime()
basic.showLeds(`
    . . # . .
    . . # . .
    . . # # #
    . . . . .
    . . . . .
    `)
while (!BWasPressed) {
    basic.pause(100)
}
BWasPressed = false
playOneGame(gameTime)
showFinalScores(scoreA, scoreB)

function startIOMonitor() {
    input.onButtonPressed(Button.A, () => {
        AWasPressed = true
    })
    input.onButtonPressed(Button.B, () => {
        BWasPressed = true
    })
    input.onButtonPressed(Button.AB, () => {
        ABWasPressed = true
        AWasPressed = false
        BWasPressed = false
    })
    input.onShake(() => {
        wasShake = true
    })
    AWasPressed = false
    BWasPressed = false
    ABWasPressed = false
    wasShake = false
}

/**
 * display score for A and B on same screen as a graphic
 * this shows a tug of war line, in the middle if scores the same,
 * Can cope with differences +/-10
 * @param scoreA1 TODO
 * @param scoreB1 TODO
 */
function showScore(scoreA1: number, scoreB1: number) {
    let img = images.createImage(`
        # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   . # . . .   . # . . .   . # . . .   . # . . .   . # . . .   . . # . .   . . . # .   . . . # .   . . . # .   . . . # .   . . . # .   . . . . #   . . . . #   . . . . #   . . . . #   . . . . #
        # . . . .   # . . . .   # . . . .   # . . . .   . # . . .   . # . . .   . # . . .   . # . . .   . # . . .   . . # . .   . . # . .   . . # . .   . . . # .   . . . # .   . . . # .   . . . # .   . . . # .   . . . . #   . . . . #   . . . . #   . . . . #
        # . . . .   # . . . .   # . . . .   . # . . .   . # . . .   . # . . .   . # . . .   . # . . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . . # .   . . . # .   . . . # .   . . . # .   . . . # .   . . . . #   . . . . #   . . . . #
        # . . . .   # . . . .   . # . . .   . # . . .   . # . . .   . # . . .   . # . . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . . # .   . . . # .   . . . # .   . . . # .   . . . # .   . . . . #   . . . . #
        # . . . .   . # . . .   . # . . .   . # . . .   . # . . .   . # . . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . . # .   . . . # .   . . . # .   . . . # .   . . . # .   . . . . #
        `)
    let diff = Math.clamp(-10, 10, scoreB1 - scoreA1)
    diff = diff + 10
    img.plotFrame(diff)
}

/**
 * show digits 0..10
 * @param digits TODO
 */
function showDigits(digits: number) {
    digits = Math.clamp(0, 10, digits)
    let img = images.createImage(`
        . . # . .   . . # . .   . # # . .   . # # . .   . # . . .   . # # # .   . . # # .   . # # # .   . . # . .   . . # . .   # . . # .
        . # . # .   . # # . .   . . . # .   . . . # .   . # . . .   . # . . .   . # . . .   . . . # .   . # . # .   . # . # .   # . # . #
        . # . # .   . . # . .   . . # . .   . . # . .   . # # # .   . . # # .   . # # . .   . . # . .   . . # . .   . . # # .   # . # . #
        . # . # .   . . # . .   . # . . .   . . . # .   . . # . .   . . . # .   . # . # .   . # . . .   . # . # .   . . . # .   # . # . #
        . . # . .   . # # # .   . # # # .   . # # . .   . . # . .   . # # . .   . . # . .   . # . . .   . . # . .   . # # . .   # . . # .
        `)
    img.plotFrame(digits)
}

/**
 * show time graphic for time remaining
 * @param gameTime TODO
 */
function showTime(gameTime: number) {
    let minutes = Math.clamp(0, 10, gameTime / 60)
    let seconds = gameTime % 60
    // divide seconds into 10 second stripes
    let stripes = seconds / 10
    let img = images.createImage(`
        . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
        . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
        . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
        . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #
        . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #
        `)
    img.plotFrame(stripes)
    // leave middle row blank
    // display up to 10 dots in raster on top two rows
    if (minutes > 0) {
        for (let i = 0; i < minutes; i++) {
            let y = i / 5
            let x = i % 5
            led.plot(x, y)
        }
    }
}

function getGameTime(): number {
    let chosenGameTime = 7
    showDigits(chosenGameTime)
    while (!BWasPressed) {
        if (AWasPressed) {
            if (chosenGameTime < 10) {
                chosenGameTime = chosenGameTime + 1
            } else {
                chosenGameTime = 1
            }
            showDigits(chosenGameTime)
            AWasPressed = false
        } else {
            basic.pause(100)
        }
    }
    BWasPressed = false
    return chosenGameTime
}

function playOneGame(gameTime: number) {
    let gameStartTime = input.runningTime()
    let gameElapsedTime = 0
    let gameTimeRemaining = gameTime * 60
    let timeout = 0
    let lastDisplayedTime = 0
    showScore(scoreA, scoreB)
    let state = "TIME"
    while (gameTimeRemaining >= 0) {
        // Tick the game time
        gameElapsedTime = (input.runningTime() - gameStartTime) / 1000
        gameTimeRemaining = gameTime * 60 - gameElapsedTime
        // Handle any global events such as point buttons
        if (AWasPressed) {
            AWasPressed = false
            scoreA = scoreA + 1
            if (state != "LAST10") {
                showScore(scoreA, scoreB)
                state = "SCORE"
            }
        } else if (BWasPressed) {
            BWasPressed = false
            scoreB = scoreB + 1
            if (state != "LAST10") {
                showScore(scoreA, scoreB)
                state = "SCORE"
            }
        }
        // Handle global transitions
        if (gameTimeRemaining <= 10 && state != "LAST10") {
            state = "LAST10"
        }
        // Handle game states
        if (state == "SCORE") {
            if (wasShake) {
                wasShake = false
                showTime(gameTimeRemaining)
                lastDisplayedTime = gameTimeRemaining
                timeout = input.runningTime() + 5 * 1000
                state = "TIME"
            }
        } else if (state == "TIME") {
            if (input.runningTime() > timeout) {
                showScore(scoreA, scoreB)
                state = "SCORE"
            }
        } else if (state == "LAST10") {
            if (gameTimeRemaining != lastDisplayedTime) {
                showDigits(gameTimeRemaining)
                lastDisplayedTime = gameTimeRemaining
            }
        }
        basic.pause(100)
    }
}

function showFinalScores(scoreA1: number, scoreB1: number) {
    basic.showLeds(`
        # . . . #
        . # . # .
        . . # . .
        . # . # .
        # . . . #
        `)
    while (true) {
        if (AWasPressed) {
            basic.showString("A", 150)
            basic.showNumber(scoreA1, 150)
            basic.showLeds(`
                # . . . #
                . # . # .
                . . # . .
                . # . # .
                # . . . #
                `)
            AWasPressed = false
        } else if (BWasPressed) {
            basic.showString("B", 150)
            basic.showNumber(scoreB1, 150)
            basic.showLeds(`
                # . . . #
                . # . # .
                . . # . .
                . # . # .
                # . . . #
                `)
            BWasPressed = false
        } else {
            basic.pause(100)
        }
    }
}

```

```blocks
let speed: number
let cupCapacity: number
let maxMisses: number
let autoEmpty: boolean
let movement: boolean
let sensitivity: number
let cupX: number
let cupInverted: boolean

let highscore = 0
while (true) {
    // configure game settings
    // ##CHALLENGE1: reconfigure game
    cupCapacity = 5
    speed = 6
    maxMisses = 3
    autoEmpty = false
    movement = true
    sensitivity = 400
    cupX = 2
    // show the spash screen
    // ##CHALLENGE 2: CHANGE SPLASH SCREEN
    basic.showAnimation(`
        . . . . .   . . . . .
        . . . . .   . # . # .
        . . . . .   . . . . .
        . # . # .   . # . # .
        . # # # .   . # # # .
        `, 400)
    // Decide what to do based on which button is pressed
    if (input.buttonIsPressed(Button.A)) {
        let finalScore = playGame()
        // ##CHALLENGE 3 ADD HIGH SCORE
        if (finalScore > highscore) {
            basic.showString("HIGH", 150)
            highscore = finalScore
        }
        basic.showNumber(finalScore, 150)
    } else if (input.buttonIsPressed(Button.B)) {
        testMovement()
    } else {
        basic.pause(100)
    }
}

function playGame(): number {
    let cup = images.createImage(`
        . . . . .   . . . . .   . . . . .
        . . . . .   . . . . .   . . . . .
        . . . . .   . . . . .   . . . . .
        . . . . .   . # . # .   . . . . .
        . . . . .   . # # # .   . . . . .
        `)
    let score = 0
    let dropsInCup = 0
    let misses = 0
    let dropX = 0
    let dropY = 0
    let prevDropY = -1
    let cupX1 = 2
    let prevCupX = -1
    let state = "NEWDROP"
    startGame()
    while (true) {
        if (state == "NEWDROP") {
            // create a new drop at a random position
            dropX = Math.random(5)
            dropY = 0
            state = "RAINING"
        } else if (state == "RAINING") {
            // calculate new positions
            cupX1 = getCupPosition()
            let thisDropY = dropY / speed
            // Only redraw the screen if something has changed (prevent flashing)
            if (cupX1 != prevCupX || thisDropY != prevDropY) {
                basic.clearScreen()
                // draw cup
                cup.showImage(7 - cupX1)
                if (dropsInCup == cupCapacity) {
                    // a full cup
                    led.plot(cupX1, 3)
                }
                // draw drop
                led.plot(dropX, thisDropY)
                prevCupX = cupX1
                prevDropY = thisDropY
            }
            basic.pause(100)
            if (thisDropY >= 4) {
                state = "ATCUP"
            } else {
                dropY = dropY + 1
            }
            if (cupInverted && dropsInCup >= cupCapacity) {
                state = "EMPTYING"
            }
        } else if (state == "ATCUP") {
            if (dropX == cupX1) {
                state = "CATCH"
            } else {
                state = "MISS"
            }
        } else if (state == "MISS") {
            // ##CHALLENGE: long beep on miss
            beep(500)
            misses = misses + 1
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .   . . . . .
                . . . . .   . # . # .   . . . . .   . . . . .
                . . . . .   . # # # .   . # . # .   . . . . .
                . # . # .   . . . . .   . # # # .   . # . # .
                . # # # .   . . # . .   . . . . .   . # # # .
                `, 400)
            if (misses > maxMisses) {
                state = "GAMEOVER"
            } else {
                state = "NEWDROP"
            }
        } else if (state == "CATCH") {
            // ##CHALLENGE: short beep on catch
            beep(200)
            dropsInCup = dropsInCup + 1
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . . . . .
                . # # # .   # . # . #   . # . # .
                . # # # .   # # # # #   . # # # .
                `, 400)
            if (dropsInCup == cupCapacity) {
                state = "FULL"
                score = score + 1
            } else if (dropsInCup > cupCapacity) {
                state = "OVERFLOW"
            } else {
                score = score + 1
                state = "NEWDROP"
            }
        } else if (state == "FULL") {
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . . . . .
                . . . . .   . # # # .   . . . . .
                . # # # .   . # # # .   . # # # .
                . # # # .   . # # # .   . # # # .
                `, 400)
            if (autoEmpty) {
                state = "EMPTYING"
            } else {
                state = "NEWDROP"
            }
        } else if (state == "EMPTYING") {
            if (cupInverted) {
                basic.showAnimation(`
                    . . . . .   . . . . .   . . . . .   . . # . .   . . . . .
                    . . . . .   . . . . .   . . # . .   . . . . .   . . . . .
                    . . . . .   . . # . .   . . . . .   . . . . .   . . . . .
                    . # # # .   . # . # .   . # . # .   . # . # .   . # . # .
                    . # # # .   . # # # .   . # # # .   . # # # .   . # # # .
                    `, 400)
            } else {
                basic.showAnimation(`
                    . . . . .   . . . . .   . # # # .   . # # # .   . # # # .   . # # # .   . # # # .   . . . . .   . . . . .
                    . . . . .   . # # . .   . # # # .   . # . # .   . # . # .   . # . # .   . # . # .   . # # . .   . . . . .
                    . . . . .   . # # . .   . . . . .   . . # . .   . . . . .   . . . . .   . . . . .   . # . . .   . . . . .
                    . # # # .   . # # . .   . . . . .   . . . . .   . . # . .   . . . . .   . . . . .   . # # . .   . # . # .
                    . # # # .   . . . . .   . . . . .   . . . . .   . . . . .   . . # . .   . . . . .   . . . . .   . # # # .
                    `, 400)
            }
            dropsInCup = 0
            // ##CHALLENGE: Speed up on every level change
            if (speed > 1) {
                speed = speed - 1
            }
            state = "NEWDROP"
        } else if (state == "OVERFLOW") {
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                . . . . .   . # # # .   . # . # .   . . . . .   . . . . .   . . . . .
                . # # # .   . # # # .   # # # # #   # # # # #   . # # # .   . # # # .
                . # # # .   . # # # .   . # # # .   . # # # .   # # # # #   . # # # .
                `, 400)
            state = "GAMEOVER"
        } else if (state == "GAMEOVER") {
            // ##CHALLENGE: Make a sound on game over
            beep(700)
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . # # # .   # # # # #   . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . # # # .   # # # # #   . # . # .   . . . . .   . . . . .   . # . # .   . . . . .
                . . . . .   . . . . .   . . . . .   . . . . .   . # # # .   # # # # #   . # . # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                . # . # .   . . . . .   . # # # .   # # # # #   . # . # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                . # # # .   # # # # #   . # . # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                `, 400)
            break
        }
    }
    return score
}

function testMovement() {
    while (!input.buttonIsPressed(Button.A)) {
        // ##CHALLENGE 5: Add accelerometer test mode
        let x = getCupPosition()
        basic.clearScreen()
        led.plot(x, 2)
        basic.pause(200)
    }
}

function getCupPosition(): number {
    if (movement) {
        let acc = input.acceleration(Dimension.X) / sensitivity
        cupX = Math.clamp(0, 4, acc + 2)
        return cupX
    }
    if (input.buttonIsPressed(Button.A)) {
        if (cupX > 0) {
            cupX = cupX - 1
        }
    } else if (input.buttonIsPressed(Button.B)) {
        if (cupX < 4) {
            cupX = cupX + 1
        }
    }
    return cupX
}

function startGame() {
    basic.clearScreen()
    // If button still held from start-game, wait until it is released
    while (input.buttonIsPressed(Button.A)) {
        // wait for button to be released
    }
    // handlers that work out if cup is turned upside down or not
    input.onLogoDown(() => {
        cupInverted = true
    })
    input.onLogoUp(() => {
        cupInverted = false
    })
}

function beep(p: number) {
    pins.digitalWritePin(DigitalPin.P0, 1)
    basic.pause(p)
    pins.digitalWritePin(DigitalPin.P0, 0)
}

```

```blocks
let AWasPressed: boolean
let BWasPressed: boolean
let ABWasPressed: boolean
let wasShake: boolean

startIOMonitor()
let gameNo = 0
while (true) {
    // select a new game
    if (AWasPressed) {
        gameNo = (gameNo + 1) % 4
        AWasPressed = false
    }
    // show selected game
    if (gameNo == 0) {
        basic.showLeds(`
            # # # # #
            # . . . #
            . # . # .
            . . . . .
            . # # # .
            `)
    } else if (gameNo == 1) {
        basic.showLeds(`
            # # # . .
            # . . . .
            # # # # #
            # # . . .
            # # # . .
            `)
    } else if (gameNo == 2) {
        basic.showLeds(`
            . . . . .
            # . . . .
            # # # # #
            # . . . .
            . . . . .
            `)
    } else if (gameNo == 3) {
        basic.showLeds(`
            # . # # #
            # # # # #
            . . . # #
            # # # # #
            # # # # #
            `)
    }
    // start selected game
    if (BWasPressed) {
        // Play the selected game
        basic.clearScreen()
        waitBReleased()
        resetButtons()
        let finalScore = 0
        basic.clearScreen()
        if (gameNo == 0) {
            finalScore = playCybermen()
        } else if (gameNo == 1) {
            finalScore = playDalek()
        } else if (gameNo == 2) {
            finalScore = playSonicScrewdriver()
        } else if (gameNo == 3) {
            finalScore = playJudoonLanguage()
        }
        flashDigit(finalScore, 3)
        resetButtons()
        waitBPressed()
        basic.clearScreen()
        waitBReleased()
        resetButtons()
    } else {
        basic.pause(100)
    }
}

/**
 * Game parameters
 * Percentage chance that cyberman will move left/right to line up with you
 */
function playCybermen(): number {
    let maxGameTime = 60
    let cybermanMoveXProbability = 20
    // How long (in 100ms ticks) cyberman must line up before moving forward
    let cybermanMoveYCount = 10
    // Game variables
    let startTime = input.runningTime()
    let gameTime = 0
    let playerX = 2
    let cybermanX = 1
    let cybermanY = 0
    let cybermanLineupCount = 0
    let redraw = true
    while (gameTime < maxGameTime) {
        if (redraw) {
            basic.clearScreen()
            led.plot(playerX, 4)
            led.plot(cybermanX, cybermanY)
            redraw = false
        }
        // Handle Player Movement
        if (AWasPressed) {
            if (playerX > 0) {
                playerX = playerX - 1
                redraw = true
            }
            AWasPressed = false
        } else if (BWasPressed) {
            if (playerX < 4) {
                playerX = playerX + 1
                redraw = true
            }
            BWasPressed = false
        }
        // Handle Cyberman line-of-sight checking
        if (cybermanX == playerX) {
            cybermanLineupCount = cybermanLineupCount + 1
            if (cybermanLineupCount >= cybermanMoveYCount) {
                if (cybermanY == 4) {
                    // Cyberman caught you, game over
                    break
                } else {
                    cybermanY = cybermanY + 1
                    redraw = true
                    cybermanLineupCount = 0
                }
            }
        } else {
            cybermanLineupCount = 0
            // Move Cyberman closer to player, slowly
            if (Math.random(100) < cybermanMoveXProbability) {
                if (cybermanX > playerX) {
                    cybermanX = cybermanX - 1
                } else if (cybermanX < playerX) {
                    cybermanX = cybermanX + 1
                }
                redraw = true
            }
        }
        basic.pause(100)
        gameTime = (input.runningTime() - startTime) / 1000
    }
    return convertSurvivalTimeToScore(gameTime)
}

/**
 * Game parameters, all probabilities as a percentage
 */
function playDalek(): number {
    let maxGameTime = 60
    let userMoveSensitivity = 40
    let dalekMoveSensitivity = 20
    let dalekShootChance = 10
    let dalekHitChance = 10
    // Game variables
    let gameTime = 0
    let startTime = input.runningTime()
    let dalekY = 2
    let userY = 1
    let redraw = true
    while (gameTime < maxGameTime) {
        // Redraw screen if necessary
        if (redraw) {
            basic.clearScreen()
            led.plot(0, dalekY)
            led.plot(4, userY)
            redraw = false
        }
        // Work out if the user has moved, and move them
        let tilt = getTilt()
        if (tilt > 2) {
            // Moving up, slowly
            if (userY < 4) {
                if (Math.random(100) < userMoveSensitivity) {
                    userY = userY + 1
                    redraw = true
                }
            }
        } else if (tilt < 2) {
            // Moving down (slowly)
            if (userY > 0) {
                if (Math.random(100) < userMoveSensitivity) {
                    userY = userY - 1
                    redraw = true
                }
            }
        }
        // Move the Dalek to line up with user
        if (dalekY < userY) {
            if (Math.random(100) < dalekMoveSensitivity) {
                dalekY = dalekY + 1
                redraw = true
            }
        } else if (dalekY > userY) {
            if (Math.random(100) < dalekMoveSensitivity) {
                dalekY = dalekY - 1
                redraw = true
            }
        } else {
            // Dalek lines up
            if (Math.random(100) < dalekShootChance) {
                // Shoot a raygun at the user
                for (let i = 0; i < 3; i++) {
                    led.plot(i + 1, dalekY)
                    basic.pause(100)
                }
                if (Math.random(100) < dalekHitChance) {
                    // User has been hit, game over
                    break
                }
                redraw = true
            }
        }
        gameTime = (input.runningTime() - startTime) / 1000
        basic.pause(100)
    }
    return convertSurvivalTimeToScore(gameTime)
}

/**
 * Set this in steps of 60
 */
function playSonicScrewdriver(): number {
    let maxGameTime = 120
    let gameTime = 0
    // @=0, A=1 etc
    // bit0=N, bit1=E, bit2=S, bit3=W
    // bit=0 means it is a wall (can not be opened)
    // bit=1 means it is a door (can be opened)
    let mazestr = "BLFL@GOIFIGLCJIA"
    let maze = ([] as number[])
    // Locks use same number encoding (bits 0,1,2,3)
    // bit=0 means door is locked (cannot be walked through)
    // bit=1 means door is open (can be walked through)
    let doorOpen = ([] as number[])
    for (let i = 0; i < 16; i++) {
        doorOpen.push(0)
        maze.push(mazestr.charCodeAt(i) - 64)
    }
    let redraw = true
    let cellno = 0
    let direction = "N"
    let startTime = input.runningTime()
    while (gameTime < maxGameTime) {
        // Draw the screen
        if (redraw) {
            basic.clearScreen()
            // Always draw the maze with N at the top
            drawMazeCell(doorOpen[cellno])
            // draw user standing next to selected wall
            if (direction == "N") {
                led.plot(2, 1)
            } else if (direction == "E") {
                led.plot(3, 2)
            } else if (direction == "S") {
                led.plot(2, 3)
            } else if (direction == "W") {
                led.plot(1, 2)
            }
            redraw = false
        }
        // Sense any button presses
        if (AWasPressed) {
            if (direction == "N") {
                direction = "E"
            } else if (direction == "E") {
                direction = "S"
            } else if (direction == "S") {
                direction = "W"
            } else if (direction == "W") {
                direction = "N"
            }
            redraw = true
            AWasPressed = false
        } else if (BWasPressed) {
            // Try to walk through an open door
            if (isDoorOpen(doorOpen[cellno], direction)) {
                cellno = mazeForward(cellno, 4, 4, direction)
                redraw = true
            }
            BWasPressed = false
        } else if (wasShake) {
            // energise sonic screwdriver
            if (isDoorOpen(maze[cellno], direction)) {
                // It is a door, but is the door open or closed?
                if (!(isDoorOpen(doorOpen[cellno], direction))) {
                    // Open the door in front of us
                    doorOpen[cellno] = openDoor(doorOpen[cellno], direction)
                    // Also open the return door for when we walk through this door
                    let forwardCellno = mazeForward(cellno, 4, 4, direction)
                    doorOpen[forwardCellno] = openDoor(doorOpen[forwardCellno], opposite(direction))
                }
            } else {
                // Not a door
                basic.showLeds(`
                    . . . . .
                    . # . # .
                    . . # . .
                    . # . # .
                    . . . . .
                    `)
                basic.pause(500)
            }
            redraw = true
            wasShake = false
        }
        if (cellno == 15) {
            // Have reached the exit cell
            break
        }
        basic.pause(100)
        gameTime = (input.runningTime() - startTime) / 1000
    }
    return convertPenaltyTimeToScore(gameTime / maxGameTime / 60)
}

function playJudoonLanguage(): number {
    let maxGameTime = 60
    let gameTime = 0
    let startTime = input.runningTime()
    let font = images.createImage(`
        # . # # #   # . # . #   . . . . #   # # . . .   # # # # #   # . # . #   # # # . .   . . . # #   # . . . #   # # # # #
        # # # # #   # # # # #   . . . # #   . # # # #   . # . . .   # # # # #   # # # # #   # # # # .   # # # # #   . . # . .
        . . . . #   # # # . .   . . # # #   . . . . #   . # . . .   . . . . #   . . . # #   . . # # #   . . # . .   # # # # #
        # # # # #   # # # # #   . . # . .   # # # # #   # # # # #   # # # # #   # # # # .   # # # # #   # # # # #   # # # . .
        # # # # #   # # # # #   # # # # #   # . . . .   # . . . #   # . . . .   . . . # #   # # # . .   # # # # #   # # # # #
        `)
    let answers = images.createImage(`
        # # # # #   # # # . .   . # # # .   . . # . .   # . # . #
        # . . . #   # . . . .   # # # # #   . . . . .   . . . . .
        . # . # .   # # # # .   # . # . #   . # . # .   # . # . #
        . . . . .   # # # . .   # . # . #   . . . . .   . . . . .
        . # # # .   # # # . .   # . # . #   # # # # #   # . . . #
        `)
    let pages = "029 041 167 208 283"
    let actualAnswer = Math.random(5)
    let pos = 0
    let redraw = true
    while (gameTime < maxGameTime) {
        // Draw the current frame from pos (0,1,2 codeword, 3,4,5,6,7 answers)
        if (redraw) {
            if (pos <= 2) {
                // Draw codeword symbol for this choice and this position
                let digit = parseInt(pages[actualAnswer * 4 + pos])
                font.plotFrame(digit)
            } else {
                // Draw answer
                let item = pos - 3
                answers.plotFrame(item)
            }
            redraw = false
        }
        // Process button presses
        if (AWasPressed) {
            // Move left, unless at far left already
            AWasPressed = false
            if (pos > 0) {
                pos = pos - 1
                redraw = true
            }
        } else if (BWasPressed) {
            // Move right, unless already at far right
            BWasPressed = false
            if (pos < 7) {
                pos = pos + 1
                redraw = true
            }
        } else if (wasShake) {
            // User trying to select an answer, are we at an answer position?
            wasShake = false
            if (pos >= 3) {
                // Is it the right answer?
                let userChoice = pos - 3
                if (userChoice == actualAnswer) {
                    // CORRECT
                    basic.showAnimation(`
                        . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                        . . . . .   . . . . .   . . . . .   . . . . .   . . . . #
                        . . . . .   . . . . .   . . . . .   . . . # .   . . . # .
                        # . . . .   # . . . .   # . # . .   # . # . .   # . # . .
                        . . . . .   . # . . .   . # . . .   . # . . .   . # . . .
                        `, 400)
                    basic.pause(1000)
                    break
                } else {
                    // WRONG
                    basic.showAnimation(`
                        . . . . .   . . . . .   . . . . .   . . . . .   . . . . #   # . . . #   # . . . #   # . . . #   # . . . #
                        . . . . .   . . . . .   . . . . .   . . . # .   . . . # .   . . . # .   . # . # .   . # . # .   . # . # .
                        . . . . .   . . . . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .
                        . . . . .   . # . . .   . # . . .   . # . . .   . # . . .   . # . . .   . # . . .   . # . # .   . # . # .
                        # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . #
                        `, 400)
                    basic.pause(1000)
                    redraw = true
                }
            }
        }
        gameTime = (input.runningTime() - startTime) / 1000
        basic.pause(100)
    }
    return convertPenaltyTimeToScore(gameTime)
}

function getDirection(): string {
    let bearing = input.compassHeading()
    if (bearing < 45 || bearing > 315) {
        return "N"
    } else if (bearing < 135) {
        return "E"
    } else if (bearing < 225) {
        return "S"
    } else {
        return "W"
    }
}

function calibrateCompass() {
    if (input.compassHeading() == -4) {
        input.calibrate()
    }
}

function waitBReleased() {
    while (input.buttonIsPressed(Button.B)) {
        basic.pause(100)
    }
}

function waitBPressed() {
    while (!input.buttonIsPressed(Button.B)) {
        basic.pause(100)
    }
}

/**
 * Show the score 0..9
 * @param digit TODO
 * @param times TODO
 */
function flashDigit(digit: number, times: number) {
    digit = Math.clamp(0, 9, digit)
    for (let i = 0; i < times; i++) {
        basic.showNumber(digit, 0)
        basic.pause(500)
        basic.clearScreen()
        basic.pause(500)
    }
    basic.showNumber(digit, 0)
}

/**
 * score is calculated as the amount of time you lasted
 * @param gameTime TODO
 */
function convertSurvivalTimeToScore(gameTime: number): number {
    if (gameTime <= 4) {
        return 0
    } else if (gameTime <= 9) {
        return 1
    } else if (gameTime <= 14) {
        return 2
    } else if (gameTime <= 19) {
        return 3
    } else if (gameTime <= 24) {
        return 4
    } else if (gameTime <= 29) {
        return 5
    } else if (gameTime <= 39) {
        return 6
    } else if (gameTime <= 49) {
        return 7
    } else if (gameTime <= 59) {
        return 8
    } else {
        return 9
    }
}

function convertPenaltyTimeToScore(penaltyTime: number): number {
    if (penaltyTime <= 4) {
        return 9
    } else if (penaltyTime <= 9) {
        return 8
    } else if (penaltyTime <= 14) {
        return 7
    } else if (penaltyTime <= 19) {
        return 6
    } else if (penaltyTime <= 24) {
        return 5
    } else if (penaltyTime <= 29) {
        return 4
    } else if (penaltyTime <= 39) {
        return 3
    } else if (penaltyTime <= 49) {
        return 2
    } else if (penaltyTime <= 59) {
        return 1
    } else {
        return 0
    }
}

function startIOMonitor() {
    input.onButtonPressed(Button.A, () => {
        AWasPressed = true
    })
    input.onButtonPressed(Button.B, () => {
        BWasPressed = true
    })
    input.onButtonPressed(Button.AB, () => {
        ABWasPressed = true
    })
    input.onShake(() => {
        wasShake = true
    })
    AWasPressed = false
    BWasPressed = false
    ABWasPressed = false
    wasShake = false
}

/**
 * maze is always drawn with north at top
 * @param cell TODO
 */
function drawMazeCell(cell: number) {
    let n = !(isNorth(cell))
    let e = !(isEast(cell))
    let s = !(isSouth(cell))
    let w = !(isWest(cell))
    // Draw any visible walls
    if (n) {
        for (let i = 0; i < 5; i++) {
            led.plot(i, 0)
        }
    }
    if (e) {
        for (let l = 0; l < 5; l++) {
            led.plot(4, l)
        }
    }
    if (s) {
        for (let k = 0; k < 5; k++) {
            led.plot(k, 4)
        }
    }
    if (w) {
        for (let j = 0; j < 5; j++) {
            led.plot(0, j)
        }
    }
}

/**
 * work out the cell number in front of this cell
 * given the direction N E S W (N points to button B)
 * returns the forward cell number, -1 if outside of maze
 * Turn cellno into an x and y based on width and height
 * @param cellno TODO
 * @param width TODO
 * @param height TODO
 * @param direction TODO
 */
function mazeForward(cellno: number, width: number, height: number, direction: string): number {
    let y = cellno / width
    let x = cellno % width
    // Work out change in x/y and therefore change in cellno
    // But bounds-check against width and height
    // as user cannot walk outside of the maze
    if (direction == "N") {
        // sub 1 from y
        if (y > 0) {
            return cellno - width
        }
    } else if (direction == "E") {
        // Add 1 to x
        if (x < width - 1) {
            return cellno + 1
        }
    } else if (direction == "S") {
        // add 1 to y
        if (y < height - 1) {
            return cellno + width
        }
    } else if (direction == "W") {
        // sub 1 from x
        if (x > 0) {
            return cellno - 1
        }
    }
    // Not allowed to move in this direction, it will go outside of maze
    return - 1
}

/**
 * A door is open if the lock bit is 1
 * A door is present if the maze bit is 1
 * @param cell TODO
 * @param direction TODO
 */
function isDoorOpen(cell: number, direction: string): boolean {
    if (direction == "N") {
        return isNorth(cell)
    } else if (direction == "E") {
        return isEast(cell)
    }
    if (direction == "S") {
        return isSouth(cell)
    } else if (direction == "W") {
        return isWest(cell)
    }
    return false
}

function getTilt(): number {
    let tilt: number
    tilt = input.acceleration(Dimension.Y)
    tilt = Math.clamp(-1024, 1023, tilt)
    tilt = (tilt + 1024) / 512
    return tilt
}

function waitAReleased() {
    while (input.buttonIsPressed(Button.A)) {
        basic.pause(100)
    }
}

function waitNoButtons() {
    while (input.buttonIsPressed(Button.A) || input.buttonIsPressed(Button.B) || input.buttonIsPressed(Button.AB)) {
        basic.pause(100)
    }
}

function resetButtons() {
    AWasPressed = false
    BWasPressed = false
    ABWasPressed = false
}

/**
 * The appropriate bit (0,1,2,3) is set, which unlocks the door
 * @param cell TODO
 * @param direction TODO
 */
function openDoor(cell: number, direction: string): number {
    if (direction == "N") {
        return cell | 1
    } else if (direction == "E") {
        return cell | 2
    } else if (direction == "S") {
        return cell | 4
    } else if (direction == "W") {
        return cell | 8
    }
    return cell
}

/**
 * is the north bit set in the cell?
 * @param cell TODO
 */
function isNorth(cell: number): boolean {
    if ((cell & 1) != 0) {
        return true
    }
    return false
}

/**
 * is the east bit set in the cell?
 * @param cell TODO
 */
function isEast(cell: number): boolean {
    if ((cell & 2) != 0) {
        return true
    }
    return false
}

/**
 * is the south bit set in the cell?
 * @param cell TODO
 */
function isSouth(cell: number): boolean {
    if ((cell & 4) != 0) {
        return true
    }
    return false
}

/**
 * is the west bit set in the cell?
 * @param cell TODO
 */
function isWest(cell: number): boolean {
    if ((cell & 8) != 0) {
        return true
    }
    return false
}

function opposite(direction: string): string {
    if (direction == "N") {
        return "S"
    } else if (direction == "E") {
        return "W"
    } else if (direction == "S") {
        return "N"
    } else if (direction == "W") {
        return "E"
    }
    return direction
}

function isSount() { }

```

```blocks
let sensorType: string

// ANALOG or TILT
sensorType = "TILT"
while (true) {
    // splash screen (flood monitor)
    basic.showLeds(`
        # # # . .
        # . . . .
        # # . # .
        # . . # .
        # . . # #
        `)
    while (true) {
        if (input.buttonIsPressed(Button.A)) {
            // test that the sensor works
            testMode()
            break
        } else if (input.buttonIsPressed(Button.B)) {
            // run the real flood monitor
            floodMonitor()
            break
        } else {
            basic.pause(100)
        }
    }
}

/**
 * test mode - test that the monitor and buzzer work
 * no filtering in this mode, direct screen update
 */
function testMode() {
    basic.showLeds(`
        # # # # #
        . . # . .
        . . # . .
        . . # . .
        . . # . .
        `)
    waitNoButtons()
    let img = images.createImage(`
        . . . . .   . . . . .   . . . . .   . . . . .   # # # # #
        . . . . .   . . . . .   . . . . .   # # # # #   . . . . .
        . . . . .   . . . . .   # # # # #   . . . . .   . . . . .
        . . . . .   # # # # #   . . . . .   . . . . .   . . . . .
        # # # # #   . . . . .   . . . . .   . . . . .   . . . . .
        `)
    while (!input.buttonIsPressed(Button.A)) {
        // Show live reading on display
        let value = readSensor()
        img.showImage(value * 5)
        // Turn beeper on only if maximum value seen
        if (value >= 4) {
            pins.digitalWritePin(DigitalPin.P1, 1)
        } else {
            pins.digitalWritePin(DigitalPin.P1, 0)
        }
        basic.pause(50)
    }
    waitNoButtons()
}

function floodMonitor() {
    basic.showLeds(`
        # . . . #
        # # . # #
        # . # . #
        # . . . #
        # . . . #
        `)
    waitNoButtons()
    let line = images.createImage(`
        . . . . .   . . . . .   . . . . .   . . . . .   # # # # #
        . . . . .   . . . . .   . . . . .   # # # # #   . . . . .
        . . . . .   . . . . .   # # # # #   . . . . .   . . . . .
        . . . . .   # # # # #   . . . . .   . . . . .   . . . . .
        # # # # #   . . . . .   . . . . .   . . . . .   . . . . .
        `)
    let state = "SAFE"
    let averagedValue = 0
    while (!input.buttonIsPressed(Button.A)) {
        let value = readSensor()
        // Apply some 'lag' filtering to the value, so that 'bobbing' doesn't set off alarm
        // i.e. to go from 0 to 4 takes 4 times round the loop
        if (value > averagedValue) {
            // On the way up, changes in reading are slowed down
            averagedValue = averagedValue + 1
        } else if (value < averagedValue) {
            // On the way down, changes in reading happen immediately
            averagedValue = value
        }
        // Work out what to do based on the averaged value (non bobbing value)
        if (state == "SAFE") {
            if (averagedValue >= 4) {
                pins.digitalWritePin(DigitalPin.P1, 1)
                state = "ALARMING"
            } else {
                // Display raw value as a line
                line.showImage(value * 5)
                // fill in based on averaged value (long term reading)
                for (let i = 0; i < averagedValue; i++) {
                    led.plot(2, 4 - i)
                }
            }
        } else if (state == "ALARMING") {
            if (input.buttonIsPressed(Button.B)) {
                pins.digitalWritePin(DigitalPin.P1, 0)
                state = "SAFE"
            } else {
                basic.showAnimation(`
                    # # # # #   . . . . .
                    # # # # #   . . . . .
                    # # # # #   . . . . .
                    # # # # #   . . . . .
                    # # # # #   . . . . .
                    `, 400)
            }
        }
        basic.pause(300)
    }
    waitNoButtons()
}

/**
 * Read a value 0,1,2,3,4 from either ANALOG or TILT
 */
function readSensor(): number {
    let value = 0
    // Produce a sensor value in range 0..1023
    if (sensorType == "ANALOG") {
        // Input range is 0..1023
        value = pins.analogReadPin(AnalogPin.P0)
    } else if (sensorType == "TILT") {
        // Input range is -1024 (pads highest)..1023 (pads lowest)
        value = input.acceleration(Dimension.Y)
        // Emulator sometimes returns out of range values, so clamp to expected range
        value = Math.clamp(-1024, 1023, value)
        // Convert to an entirely positive range from 0 pads full up to 2047 pads full down
        // (hinged at the 'eyes' end)
        value = value + 1024
        // Invert the direction, so that "pads up" is largest, "pads down" is smallest
        value = 2047 - value
        // We want the same range as the analog, so halve the range
        value = value / 2
    }
    // 5 possible values (0,1,2,3,4) based on sensible thresholds
    // We do this by thresholds, so that we have more control over the 5 levels
    if (value < 200) {
        return 0
    } else if (value < 400) {
        return 1
    } else if (value < 600) {
        return 2
    } else if (value < 800) {
        return 3
    }
    return 4
}

function waitNoButtons() {
    while (input.buttonIsPressed(Button.A) || input.buttonIsPressed(Button.B)) {
        basic.pause(100)
    }
}

```

```blocks
let opMaxTime: number
let procMaxTime: number
let procsToDo: number
let recoveryTriesMax: number
let tenseTime: number
let flatlineTimeMax: number
let recoveryProbability: number
let aWasPressed: boolean
let bWasPressed: boolean
let digitsImg: Image
let tweezerCount_: number
let wasTweezers: boolean
let wasNose: boolean
let isTweezers: boolean

// P0 blue wire, tweezers (active high)
// P1, green wire nose button/LED (active high)
// P2/red speaker (not a self toned beeper, but a piezo or a speaker)
opMaxTime = 120 * 1000
procsToDo = 3
procMaxTime = 60 * 1000
tenseTime = 4000
flatlineTimeMax = 15 * 1000
recoveryTriesMax = 3
recoveryProbability = 60
let highScore = 0
digitsImg = images.createImage(`
    . . # . .   . . # . .   . # # . .   . # # . .   . # . . .   . # # # .   . . # # .   . # # # .   . . # . .   . . # . .
    . # . # .   . # # . .   . . . # .   . . . # .   . # . . .   . # . . .   . # . . .   . . . . .   . # . # .   . # . # .
    . # . # .   . . # . .   . . # . .   . . # . .   . # # # .   . . # . .   . # # . .   . . # . .   . . # . .   . . # # .
    . # . # .   . . # . .   . # . . .   . . . # .   . . # . .   . . . # .   . # . # .   . # . . .   . # . # .   . . . # .
    . . # . .   . # # # .   . # # # .   . # # . .   . . # . .   . # # . .   . . # . .   . # . . .   . . # . .   . # # . .
    `)
startIOMonitor()
while (true) {
    splashScreen()
    if (buttonB()) {
        basic.showAnimation(`
            . . . . .   # # . . .   . # . . .   . # . . .   . . . # .   . . . . .
            . . . . #   . . . . #   # # . . #   . # . . #   . . # . #   . . . . #
            . . . . #   . . . . #   . . . . #   # # . . #   # # . . #   # # # # #
            # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
            # . . . #   # . . . #   # . . . #   # . . . #   # . . . #   # . . . #
            `, 400)
        let finalScore = surgery()
        if (game.score() > highScore) {
            basic.showString("HI", 150)
            highScore = finalScore
        }
        flashDigit(finalScore)
        waitButtonB()
    } else if (buttonA()) {
        testMode()
    } else {
        basic.pause(100)
    }
}

function surgery(): number {
    let score = 0
    let speed = 150
    let opStartTime = input.runningTime()
    let procStartTime = -1
    let procNumber = -1
    let procsDone = 0
    let recoveryTry = 0
    let timer = 0
    let state = "CHOOSE PROC"
    resetButtons()
    while (true) {
        basic.pause(10)
        let event = getEvent(state)
        if (event == "CUT") {
            state = "ARREST"
        }
        // CHECK TIMERS
        if (!(procStartTime == -1)) {
            if (input.runningTime() - procStartTime > procMaxTime) {
                state = "LOST"
            } else {
                // TODO add code here to speed up near end of proc
            }
        }
        if (input.runningTime() - opStartTime > opMaxTime) {
            state = "LOST"
        } else {
            // TODO add code here to speed up near end of op
        }
        // PROCESS SPECIFIC STATES
        if (state == "CHOOSE PROC") {
            if (procNumber == -1) {
                procNumber = 1
                showDigit(procNumber)
            } else if (event == "SCROLL") {
                procNumber = procNumber + 1
                if (procNumber > 9) {
                    procNumber = 1
                }
                showDigit(procNumber)
            } else if (event == "SELECT") {
                procStartTime = input.runningTime()
                state = "HEALTHY"
                speed = 100
            }
        } else if (state == "HEALTHY") {
            speed = 100
            ecg(speed)
            if (event == "TWINGE") {
                state = "TENSE"
                timer = input.runningTime()
            } else if (event == "DONE") {
                state = "PROC DONE"
            }
        } else if (state == "TENSE") {
            speed = 25
            ecg(speed)
            if (event == "TWINGE") {
                state = "ARREST"
            } else if (input.runningTime() - timer > tenseTime) {
                state = "HEALTHY"
            }
        } else if (state == "ARREST") {
            timer = input.runningTime()
            recoveryTry = recoveryTriesMax
            state = "FLATLINE"
        } else if (state == "FLATLINE") {
            basic.showLeds(`
                . . . . .
                . # . # .
                . . . . .
                . . . . .
                # # # # #
                `)
            beepOn()
            if (event == "SHOCK") {
                state = "SHOCKING"
            } else if (input.runningTime() - timer > flatlineTimeMax) {
                state = "LOST"
            }
        } else if (state == "SHOCKING") {
            charging()
            basic.showAnimation(`
                . . . . #   . . . . #   . . . . .   . . . . .
                . . . # .   . . . # .   . . . # .   . . . . .
                . . . . .   . . # . .   . . # . .   . . # . .
                . . . . .   . . . . .   . # . . .   . # . . .
                . . . . .   . . . . .   . . . . .   . # . . .
                `, 150)
            beepNTimesFor(15, 500)
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .   # . . . #
                . . . . .   . . . . .   # . . # .   . . . . .
                . . . . .   # . # . .   . . . . .   . . . . .
                . # . . .   . # . . .   . . . . .   . . . . .
                . # . . .   . # . . .   . # . . .   . # . . .
                `, 150)
            state = "SHOCKED"
        } else if (state == "SHOCKED") {
            let recover = Math.random(100)
            if (recover >= recoveryProbability) {
                state = "RECOVERED"
            } else {
                state = "FAILED"
            }
        } else if (state == "RECOVERED") {
            beepOff()
            basic.pause(500)
            beep()
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                . # . # .   . . . . .   . # . # .   . . . . .   . # . # .
                . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                # . . . #   . . . . .   # . . . #   . . . . .   # . . . #
                . # # # .   . . . . .   . # # # .   . . . . .   . # # # .
                `, 400)
            state = "HEALTHY"
        } else if (state == "FAILED") {
            recoveryTry = recoveryTry - 1
            if (recoveryTry > 0) {
                state = "FLATLINE"
            } else {
                state = "LOST"
            }
        } else if (state == "PROC DONE") {
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . . . . .   . . . . .   . . . . #
                . . . . .   . . . . .   . . . . .   . . . # .   . . . # .
                # . . . .   # . . . .   # . # . .   # . # . .   # . # . .
                . . . . .   . # . . .   . # . . .   . # . . .   . # . . .
                `, 400)
            score = score + pointsForProc(procNumber)
            procsDone = procsDone + 1
            procStartTime = -1
            if (procsDone == procsToDo) {
                state = "OP DONE"
            } else {
                procNumber = -1
                state = "CHOOSE PROC"
            }
        } else if (state == "OP DONE") {
            basic.showAnimation(`
                . . . . .   . . . # .   . # . . .   . . # . .   . . . # .   . . . . #   . . # . .   . . # . .   . . . . .   . . . . .   . . . . .
                . . . . #   . . # . #   . # . . #   . . # . .   . . . # .   . . . . #   . . # . .   . . # . .   . # . . .   # . . . .   . . . . .
                # # # # #   # # . . #   # # . . #   . # # . .   . . # # .   . . . # #   . # # . .   . . # . .   . # . . .   # . . . .   . . . . .
                # # # # #   # # # # #   # # # # #   . # # # #   . . # # #   . . . # #   . . . # #   . . # # #   . # . # #   # . . # #   . . . # #
                # . . . #   # . . . #   # . . . #   . # . . .   . . # . .   . . . # .   . . . # .   . . . # .   . # . # .   # . . # .   . . . # .
                `, 400)
            return score
        } else if (state == "LOST") {
            beepOn()
            basic.showLeds(`
                . # . # .
                . . # . .
                . # . # .
                . . . . .
                # # # # #
                `)
            basic.pause(3000)
            beepOff()
            basic.showAnimation(`
                . . . . .   . . . . .   # # # # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                . . . . #   # # # # #   . . . . #   . . . . #   . . . . #   # . # . #   . . . . #   # . # . #   . . . . #
                # # # # #   . . . . #   . . . . #   . . . . #   . . . . #   . . . . #   . . . . #   . . . . #   . . . . #
                # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
                # . . . #   # . . . #   # . . . #   # . . . #   # . . . #   # . . . #   # . . . #   # . . . #   # . . . #
                `, 400)
            return score
        }
    }
}

/**
 * if any button pressed, terminate the animation immediately
 */
function splashScreen() {
    let img = images.createImage(`
        . # . # .   . # . # .   . # . # .   . # . # .   . . # . .   . # # . .
        # # # # #   # . # . #   # # # # #   # . # . #   . # . # .   . # . # .
        # # # # #   # . . . #   # # # # #   # . . . #   . # . # .   . # # . .
        . # # # .   . # . # .   . # # # .   . # . # .   . # . # .   . # . . .
        . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . # . . .
        `)
    let x = 0
    while (!aWasPressed && !bWasPressed) {
        img.showImage(x)
        basic.pause(500)
        x = x + 5
        if (x >= img.width()) {
            x = 0
        }
    }
}

/**
 * Test sensing and buzzing
 * I/O at moment is (assuming self toning beeper)
 * P0 is beeper and nose LED
 * P1 is the tweezer sense wire
 * P2 is possibly the nose button
 * If we want amplification, might have to use piezo library
 * which means using extra pins
 */
function testMode() {
    while (!(buttonA())) {
        if (pins.digitalReadPin(DigitalPin.P1) == 1) {
            pins.digitalWritePin(DigitalPin.P0, 1)
            basic.showLeds(`
                . . . . .
                . . . . #
                . . . # .
                # . # . .
                . # . . .
                `)
        } else {
            pins.digitalWritePin(DigitalPin.P0, 0)
            basic.showLeds(`
                # # # # #
                . . # . .
                . . # . .
                . . # . .
                . . # . .
                `)
        }
        basic.pause(100)
    }
}

/**
 * SENSE TWINGE/CUT FROM TWEEZERS (10ms per sample)
 * @param state TODO
 */
function getEvent(state: string): string {
    if (wasTweezers) {
        if (tweezerCount_ > 20) {
            wasTweezers = false
            tweezerCount_ = 0
            return "CUT"
        } else if (!isTweezers) {
            wasTweezers = false
            tweezerCount_ = 0
            return "TWINGE"
        }
    }
    // SENSE A OR B BUTTON PRESSES
    if (state == "CHOOSE PROC") {
        if (buttonA()) {
            return "SCROLL"
        } else if (buttonB()) {
            return "SELECT"
        }
    } else if (state == "FLATLINE") {
        if (buttonB()) {
            return "SHOCK"
        } else if (nose()) {
            return "SHOCK"
        }
    } else if (state == "HEALTHY") {
        if (buttonB()) {
            return "DONE"
        }
    }
    // Clear any flags for unnecessary events in a given state to prevent latching
    aWasPressed = false
    bWasPressed = false
    return "NONE"
}

function flashDigit(digit: number) {
    for (let i = 0; i < 4; i++) {
        showDigit(digit)
        basic.pause(200)
        basic.clearScreen()
        basic.pause(200)
    }
    showDigit(digit)
    basic.pause(1000)
}

/**
 * Make a short beep sound
 */
function beep() {
    beepOn()
    basic.pause(200)
    beepOff()
}

/**
 * work out score for a procedure
 * @param procNumber TODO
 */
function pointsForProc(procNumber: number): number {
    if (procNumber < 4) {
        return 1
    } else if (procNumber < 7) {
        return 2
    } else {
        return 3
    }
}

/**
 * beep n times, for a total duration of m
 * @param times TODO
 * @param duration TODO
 */
function beepNTimesFor(times: number, duration: number) {
    let halfCycle = duration / (times * 2)
    for (let i = 0; i < times; i++) {
        beepOn()
        basic.pause(halfCycle)
        beepOff()
        basic.pause(halfCycle)
    }
}

function startIOMonitor() {
    aWasPressed = false
    input.onButtonPressed(Button.A, () => {
        aWasPressed = true
    })
    bWasPressed = false
    input.onButtonPressed(Button.B, () => {
        bWasPressed = true
    })
    wasTweezers = false
    isTweezers = false
    tweezerCount_ = 0
    control.inBackground(() => {
        let buzzCount = 0
        while (true) {
            if (pins.digitalReadPin(DigitalPin.P0) == 1) {
                wasTweezers = true
                isTweezers = true
                tweezerCount_ = tweezerCount_ + 1
                if (buzzCount == 0) {
                    pins.analogWritePin(AnalogPin.P2, 512)
                    pins.analogSetPeriod(AnalogPin.P2, 5000)
                }
                buzzCount = 10
            } else {
                isTweezers = false
            }
            if (buzzCount > 0) {
                buzzCount = buzzCount - 1
                if (buzzCount == 0) {
                    pins.analogWritePin(AnalogPin.P2, 0)
                }
            }
            basic.pause(10)
        }
    })
}

/**
 * Shows a single digit, with a nicer font than the standard micro:bit font
 * @param digit TODO
 */
function showDigit(digit: number) {
    digit = Math.clamp(0, 9, digit)
    digitsImg.showImage(digit * 5)
}

/**
 * check to see if button A was pressed recently
 */
function buttonA(): boolean {
    if (aWasPressed) {
        aWasPressed = false
        return true
    }
    return false
}

function waitButtonA() {
    while (!(buttonA())) {
        basic.pause(100)
    }
}

function buttonB(): boolean {
    if (bWasPressed) {
        bWasPressed = false
        return true
    }
    return false
}

function waitButtonB() {
    while (!(buttonB())) {
        basic.pause(100)
    }
}

function beepOn() {
    pins.analogWritePin(AnalogPin.P2, 512)
    pins.analogSetPeriod(AnalogPin.P2, 2272)
}

function beepOff() {
    pins.analogWritePin(AnalogPin.P2, 0)
}

function resetButtons() {
    aWasPressed = false
    bWasPressed = false
}

function ecg(speed: number) {
    beepOn()
    pins.digitalWritePin(DigitalPin.P1, 1)
    basic.pause(50)
    beepOff()
    basic.showAnimation(`
        . . . . #   . . . # .   . . # . .   . # . . .   # . . . .   . . . . .   . . . . .
        . . . . #   . . . # .   . . # . .   . # . . .   # . . . .   . . . . .   . . . . .
        . . . . #   . . . # .   . . # . .   . # . . .   # . . . .   . . . . .   . . . . .
        . . . . #   . . . # #   . . # # .   . # # . .   # # . . .   # . . . .   . . . . .
        # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
        `, speed)
    pins.digitalWritePin(DigitalPin.P1, 0)
    basic.pause(speed * 10)
}

function nose(): boolean {
    if (wasNose) {
        wasNose = false
        return true
    }
    return false
}

/**
 * start period in microseconds
 */
function charging() {
    let period = 2000
    let dec = 500
    pins.analogWritePin(AnalogPin.P2, 512)
    pins.analogSetPeriod(AnalogPin.P2, period)
    basic.showLeds(`
        . # # . .
        . . . # .
        . . # . .
        . . . # .
        . # # . .
        `)
    basic.pause(500)
    pins.analogSetPeriod(AnalogPin.P2, period - dec)
    basic.showLeds(`
        . # # . .
        . . . # .
        . . # . .
        . # . . .
        . # # # .
        `)
    basic.pause(500)
    pins.analogSetPeriod(AnalogPin.P2, period - dec * 2)
    basic.showLeds(`
        . . # . .
        . # # . .
        . . # . .
        . . # . .
        . # # # .
        `)
    basic.pause(500)
    beepOff()
}

```

```blocks
let wasShake: boolean

input.onShake(() => {
    wasShake = true
})
wait_6Hours()
timeForTest()
alertnessTest()
wait_2Hours()
timeForTest()
alertnessTest()
wait_2Hours()
timeForTest()
alertnessTest()

/**
 * animate back to work
 * animate take a rest
 */
function animations() { }

/**
 * wait 6 hours
 * because this is a test program, we only wait for a short time
 */
function wait_6Hours() {
    basic.showAnimation(`
        . . # . .   . . . . #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .
        . . # . .   . . . # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . # . . .
        . . # . .   . . # . .   . . # # #   . . # . .   . . # . .   . . # . .   # # # . .   . . # . .
        . . . . .   . . . . .   . . . . .   . . . # .   . . # . .   . # . . .   . . . . .   . . . . .
        . . . . .   . . . . .   . . . . .   . . . . #   . . # . .   # . . . .   . . . . .   . . . . .
        `, 500)
}

/**
 * wait for 2 hours
 * because this is test code, we only wait a few seconds
 */
function wait_2Hours() {
    basic.showAnimation(`
        . . # . .   . . . . #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .
        . . # . .   . . . # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . # . . .
        . . # . .   . . # . .   . . # # #   . . # . .   . . # . .   . . # . .   # # # . .   . . # . .
        . . . . .   . . . . .   . . . . .   . . . # .   . . # . .   . # . . .   . . . . .   . . . . .
        . . . . .   . . . . .   . . . . .   . . . . #   . . # . .   # . . . .   . . . . .   . . . . .
        `, 500)
}

function alertnessTest() {
    let goodResponse = 1000
    let threshold = 5
    let score = 0
    // Start test on button press
    let x = 0
    let start = images.createImage(`
        . . # . .   . . # . .
        . . # . .   . # . # .
        . . # . .   . . . # .
        . . . . .   . . # . .
        . . # . .   . . # . .
        `)
    while (!input.buttonIsPressed(Button.B)) {
        start.showImage(x)
        x = x + 5
        if (x >= start.width()) {
            x = 0
        }
        basic.pause(300)
    }
    // Wait for button(s) to be released
    while (input.buttonIsPressed(Button.A) || input.buttonIsPressed(Button.B)) {
        basic.pause(100)
    }
    // Run 10 random cognition response tests
    for (let i = 0; i < 9; i++) {
        // Choose random delay and random outcome
        let delay = Math.random(5) + 5
        let outcome = Math.random(2)
        // Draw moving dots on screen until delay expires
        basic.clearScreen()
        for (let j = 0; j < delay; j++) {
            led.plot(j % 5, 2)
            basic.pause(200)
            basic.clearScreen()
        }
        // Show shake or button icon
        if (outcome == 0) {
            // Press the button!
            basic.showLeds(`
                . . . . .
                . # # # .
                . # # # .
                . # # # .
                . . . . .
                `)
        } else {
            // Shake the bit!
            basic.showLeds(`
                # . # . .
                # . . # .
                # # # # #
                # . . # .
                # . # . .
                `)
        }
        // Wait up to 3 seconds for button, shake, or timeout
        wasShake = false
        let timer = input.runningTime()
        let timeout = 3000
        while (input.runningTime() < timer + timeout) {
            if (wasShake) {
                break
            } else if (input.buttonIsPressed(Button.B)) {
                break
            } else {
                basic.pause(100)
            }
        }
        // Assess the response and the response time
        let response = input.runningTime() - timer
        if (outcome == 0 && input.buttonIsPressed(Button.B) && response <= goodResponse) {
            score = score + 1
        } else if (outcome == 1 && wasShake && response <= goodResponse) {
            score = score + 1
        }
    }
    // Show final score flashing 5 times (0..9)
    for (let k = 0; k < 5; k++) {
        basic.showNumber(score, 0)
        basic.pause(250)
        basic.clearScreen()
        basic.pause(250)
    }
    basic.showNumber(score, 0)
    basic.pause(500)
    if (score < threshold) {
        // Time for a break, show coffee cup animation
        for (let l = 0; l < 3; l++) {
            basic.showAnimation(`
                . . . . .   . . . . .   . # . . .   . # . . .   . . . . .   . . # . .   . . # . .   . . . . .
                . . . . .   . # . . .   . # . . .   . . . . .   . . # . .   . . # . .   . . . . .   . . . . .
                # . . # .   # # . # .   # . . # .   # . . # .   # . # # .   # . . # .   # . . # .   # . . # .
                # . . # #   # . . # #   # . . # #   # . . # #   # . . # #   # . . # #   # . . # #   # . . # #
                # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .
                `, 400)
        }
    } else {
        // All ok, back to work, show digging animation
        for (let m = 0; m < 3; m++) {
            basic.showAnimation(`
                # . # . .   # . . . .   # . . . .
                . # # . .   . # . # .   . # . . .
                # # # . .   . . # # .   . . # . #
                . . . . .   . # # # .   . . . # #
                . . . . .   . . . . .   . . # # #
                `, 400)
        }
    }
    // Wait for any button press to finish test
    while (!input.buttonIsPressed(Button.A) && !input.buttonIsPressed(Button.B)) {
        basic.pause(100)
    }
}

/**
 * alert the user it is time to take the test
 * in a real system, this might give them 5 1 minute warnings
 */
function timeForTest() {
    basic.showAnimation(`
        . # # # .   . # . . .   . # # . .   . # # . .   . . # . .
        . # . . .   . # . . .   . . . # .   . . . # .   . # # . .
        . . # . .   . # # # .   . # # . .   . . # . .   . . # . .
        . . . # .   . . # . .   . . . # .   . # . . .   . . # . .
        . # # . .   . . # . .   . # # . .   . # # # .   . # # # .
        `, 700)
}

```

```blocks
let maxGameTime: number
let wasShake: boolean

maxGameTime = 60000
input.onShake(() => {
    wasShake = true
})
while (true) {
    splashScreen()
    if (input.buttonIsPressed(Button.B)) {
        basic.showAnimation(`
            # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # . .   # # # # #
            # # # . .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # . .   # . . . .   # . . . .   # . . . .   # . . . .   # # # # #   # # # # #
            # # # . .   # # # . .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # . .   # # # . .   # # # . .   # # # # #   # # # # #   # # # # #
            . . . . .   . . . . .   . . . . .   . . . # .   . . . # .   . . . # .   . . . # .   . . . # .   . . . # .   . . . # #   . . # # #   . # # # #   # # # # #   # # # # #   # # # # #   # # # # #
            . . . . .   . . . . .   . . . . .   . . . . .   . . . # .   . . . # #   . . # # #   . # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
            `, 400)
        let score = playOneGame()
        flashDigit(score, 5)
        waitButtonB()
        basic.showAnimation(`
            # # # # #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
            # # # # #   # # # # #   . . . . .   . . . . .   . . . . .   . . . . .   . # . # .   . . . . .   . # . # .   . . . . .
            # # # # #   # # # # #   # # # # #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
            # # # # #   # # # # #   # # # # #   # # # # #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
            # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
            `, 400)
    } else if (input.buttonIsPressed(Button.A)) {
        calibrate()
    }
}

function playOneGame(): number {
    let countDots = 0
    let x = 2
    let y = 2
    let xdir = true
    let canvas = images.createImage(`
        . . . . .
        . . . . .
        . . . . .
        . . . . .
        . . . . .
        `)
    wasShake = false
    let bearing = input.compassHeading()
    let startTime = input.runningTime()
    while (countDots < 25 && input.runningTime() - startTime < maxGameTime) {
        if (wasShake) {
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .   . . . . .   . # # # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . . . . .   . # # # .   . . # . .   . # # # .   . . . . .   . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . # # # .   . . # . .   . . # . .   . . # . .   . # # # .   . . . . .   . . . . .   . . . . .
                . . . . .   . # # # .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . # # # .   . . . . .   . . . . .
                . # # # .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . # # # .   . . . . .
                `, 50)
            let pos = Math.random(5)
            if (xdir) {
                if (!canvas.pixel(pos, y)) {
                    canvas.setPixel(pos, y, true)
                    countDots = countDots + 1
                }
            } else if (!canvas.pixel(x, pos)) {
                canvas.setPixel(x, pos, true)
                countDots = countDots + 1
            }
            wasShake = false
            canvas.showImage(0)
        } else if (Math.abs(input.compassHeading() - bearing) > 60) {
            xdir = !xdir
            bearing = input.compassHeading()
            if (xdir) {
                x = Math.random(5)
            } else {
                y = Math.random(5)
            }
        } else {
            basic.pause(100)
        }
    }
    return dotsToScore(countDots)
}

function splashScreen() {
    let img = images.createImage(`
        # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   . # # # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . # .   . . # # .   . # # # .   # # # # .   # # # # .   . # # # .   . . # # .   . . . # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .
        # . . . #   # . . . #   # . . . #   # . . . #   # . . . #   . # # # .   . . # . .   . # # # .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .   . # . . .   . . # . .   . . . # .   . . . . #   . . . # #   . . # . #   . # . . #   # . . . #   # . . . #   # . . . #   # . . . #   . # . . #   . . # . #   . . . # #   . . . . #   . . . # .   . . # . .   . # . . .   # . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . # # # .   . . # . .   . # # # .   # . . . #   # . . . #   # . . . #   # . . . #
        # # # # .   # # # # .   # # # # .   # # # # .   . # # # .   . . # . .   . . # . .   . . # . .   . # # # .   . . . . .   . . . . .   . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # .   # # # # #   # # # # .   # # # . .   # # . . .   # . . . .   . . . . .   . . . . .   . . . . .   . # # # .   . . # . .   . . # . .   . . # . .   . # # # .   # # # # .   # # # # .   # # # # .
        # . . . .   # . . . .   # . . . .   . # # # .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . # # # .   . . . . .   . . . . .   # . . . .   . # . . .   . . # . .   . . . # .   . . . . #   . . . # .   . . # # .   . # . # .   # . . # .   # . . # .   # . . # .   # . . # .   . # . # .   . . # # .   . . . # .   . . . . #   . . . # .   . . # . .   . # . . .   # . . . .   . . . . .   . . . . .   . # # # .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . # # # .   # . . . .   # . . . .
        # . . . .   # . . . .   . # # # .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . # # # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . #   . . . . #   . . . . #   . . . . #   # . . . #   # . . . #   . . . . #   . . . . #   . . . . #   . . . . #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . # # # .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . . # . .   . # # # .   # . . . .
        `)
    let x = 0
    while (!input.buttonIsPressed(Button.A) && !input.buttonIsPressed(Button.B)) {
        img.showImage(x)
        basic.pause(200)
        x = x + 5
        if (x >= img.width()) {
            x = 0
        }
    }
}

/**
 * rotate canvas
 */
function rotateImage() { }

function flashDigit(digit: number, times: number) {
    for (let i = 0; i < times; i++) {
        basic.showNumber(digit, 0)
        basic.pause(500)
        basic.clearScreen()
        basic.pause(500)
    }
    basic.showNumber(digit, 0)
}

function waitButtonB() {
    while (!input.buttonIsPressed(Button.B)) {
        basic.pause(100)
    }
}

function dotsToScore(dots: number): number {
    if (dots == 25) {
        return 9
    } else if (dots >= 22) {
        return 8
    } else if (dots >= 19) {
        return 7
    } else if (dots >= 16) {
        return 6
    } else if (dots >= 13) {
        return 5
    } else if (dots >= 10) {
        return 4
    } else if (dots >= 7) {
        return 3
    } else if (dots >= 4) {
        return 2
    } else if (dots >= 1) {
        return 1
    } else {
        return 0
    }
}

function calibrate() {
    basic.showString("CAL", 150)
    if (input.compassHeading() == -4) {
        input.calibrate()
    }
    while (!input.buttonIsPressed(Button.B)) {
        let h = input.compassHeading()
        basic.showNumber(h, 150)
    }
}

```

```blocks
let state: string
let timer: number
let gesturesRunning: boolean
let wasShake: boolean
let wasLogoUp: boolean
let wasLogoDown: boolean
let wasScreenUp: boolean
let wasScreenDown: boolean

while (true) {
    splashScreen()
    if (input.buttonIsPressed(Button.B)) {
        // animate: add pancake
        basic.showAnimation(`
            # # # # #   # . . . #   . . . . .   . . . . .   . . . . .
            . . . . .   . # # # .   . # # # .   . . . . .   . . . . .
            . . . . .   . . . . .   # . . . #   # . . . #   . . . . .
            . . . . .   . . . . .   . . . . .   . # # # .   . # # # .
            . . . . .   . . . . .   . . . . .   . . . . .   # . . . #
            `, 250)
        runGameOnce()
        // animate: pancake done
        basic.showAnimation(`
            . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . # # # .   # # # # #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
            . . . . .   . . . . .   . . . . .   . . . . .   # . . . #   # # # # #   # . . . #   . . . . .   . . . . .   . # . # .   . . . . .   . # . # .   . . . . .
            . . . . .   . . . . .   . # # # .   # # # # #   . # # # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
            # . . . #   # # # # #   # . . . #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
            . # # # .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
            `, 250)
    } else if (input.buttonIsPressed(Button.A)) {
        testShake()
    }
}

/**
 * Runs one complete game from start to end
 */
function runGameOnce() {
    let score = 0
    let cooks = 0
    let target_time = 0
    // make sure no gestures are outstanding from last game
    wasShake = false
    wasLogoUp = false
    wasLogoDown = false
    wasScreenUp = false
    wasScreenDown = false
    state = "NEWSIDE"
    while (true) {
        // Handle any gestures that can happen at any time
        let gesture = getGesture()
        if (gesture == "FLOOR") {
            state = "DROPPED"
        }
        // Run code appropriate to the present state of the game
        if (state == "NEWSIDE") {
            target_time = 5 + Math.random(5)
            state = "COOKING"
            startTimer()
        } else if (state == "COOKING") {
            // animate: cooking
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . . . . .   . . . . .
                . . . . .   # . . . .   . . . . .   . . . . #
                # # # # #   . # # # #   # # # # #   # # # # .
                `, 100)

            if (gesture == "FLIP") {
                state = "FLIPPING"
            } else if (getTimerSec() >= target_time) {
                state = "READY"
                score = score + 1
                startTimer()
            }
        } else if (state == "READY") {
            // animate: ready
            basic.showAnimation(`
                . . . . .
                . . . . .
                . . . . .
                # . . . #
                . # # # .
                `, 100)
            if (getTimerSec() > 2) {
                state = "BURNING"
                score = score - 1
                startTimer()
            } else if (gesture == "FLIP") {
                state = "FLIPPING"
            }
        } else if (state == "BURNING") {
            // animate: burning
            basic.showAnimation(`
                . . . . .   . . . . .   . . . # .   . . . # .
                . . . . .   . . . # .   . # . # .   . # . . .
                . . . . .   . # . # .   . # . . .   . . . . .
                . . . . .   . # . . .   . . . . .   . . . . .
                # # # # #   # # # # #   # # # # #   # # # # #
                `, 100)
            if (gesture == "SKY") {
                state = "NEWSIDE"
            } else if (getTimerSec() > 2) {
                state = "BURNT"
            }
        } else if (state == "FLIPPING") {
            // animate: flipping
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .   . . . . .   # . . . .   . . . . .   . . . . #   . . . . .   . . . . .   . . . . .
                . . . . .   . . . . .   . . . . .   # # # # #   . # # . .   . # # # .   . . # # .   # # # # #   . . . . .   . . . . .
                . . . . .   . . . . .   # # # # #   . . . . .   . . . # #   . # # # .   # # . . .   . . . . .   # # # # #   . . . . .
                . . . . .   # # # # #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # # # # #
                # # # # #   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
                `, 100)
            // Prevent a spurious double-flip from happening
            wasShake = false
            cooks = cooks + 1
            if (cooks == 5) {
                state = "GAMEOVER"
            } else {
                state = "NEWSIDE"
            }
        } else if (state == "DROPPED") {
            // animate: dropped
            basic.showAnimation(`
                # . . . #   . . . . .   # . . . #   . . . . .
                . # . # .   . . . . .   . # . # .   . . . . .
                . . # . .   . . . . .   . . # . .   . . . . .
                . # . # .   . . . . .   . # . # .   . . . . .
                # . . . #   . . . . .   # . . . #   . . . . .
                `, 250)
            score = 0
            state = "GAMEOVER"
        } else if (state == "BURNT") {
            // animate: burnt
            basic.showAnimation(`
                . . . . .   . . . . .   . . . . .   . . . . .   . . # . .   . . . . .   . # . # .   . . . . .   . # # # .
                . . . . .   . . . . .   . . . . .   . . # . .   . . . . .   . # . # .   . . . . .   . # # # .   . . . . .
                . . . . .   . . . . .   . . . . .   . . . . .   . # . # .   . . . . .   . # # # .   . . . . .   . . . . .
                . . . . .   . # . # .   . # # # .   . # . # .   . . . . .   . # # # .   . . . . .   . . . . .   . . . . .
                # # # # #   . # # # .   . # # # .   . # # # .   . # # # .   . . . . .   . . . . .   . . . . .   . . . . .
                `, 250)
            score = 0
            state = "GAMEOVER"
        } else if (state == "GAMEOVER") {
            animateScore(score)
            state = "WAITEJECT"
        } else if (state == "WAITEJECT") {
            if (gesture == "UPSIDEDOWN") {
                return
            }
        }
    }
}

/**
 * show score (0..9) flashing
 * @param score TODO
 */
function animateScore(score: number) {
    score = Math.clamp(0, 9, score)
    for (let i = 0; i < 5; i++) {
        basic.showNumber(score, 0)
        basic.pause(500)
        basic.clearScreen()
        basic.pause(500)
    }
    basic.showNumber(score, 0)
}

/**
 * NOTE: Eventually this will move into a gesture library
 * It hides all the nasty detail of detecting gestures
 */
function getGesture(): string {
    if (!gesturesRunning) {
        input.onShake(() => {
            wasShake = true
        })
        input.onLogoUp(() => {
            wasLogoUp = true
        })
        input.onLogoDown(() => {
            wasLogoDown = true
        })
        input.onScreenUp(() => {
            wasScreenUp = true
        })
        input.onScreenDown(() => {
            wasScreenDown = true
        })
        gesturesRunning = true
    }
    // work out which of a possible set of gestures has occurred:
    // Button gestures and movement gestures both handled
    // This is so that it is easy to also use this on the simulator too
    // Generally, B is used for normal OK, A is used for abnormal RECOVERY
    // (flip is a normal action, touch sky to turn off smoke alarm is recovery)
    let a = input.buttonIsPressed(Button.A)
    let b = input.buttonIsPressed(Button.B)
    if (state == "COOKING" || state == "READY") {
        if (b || wasShake) {
            wasShake = false
            return "FLIP"
        }
    } else if (state == "FLIPPING") {
        if (a || wasLogoDown) {
            wasLogoDown = false
            return "FLOOR"
        }
    } else if (state == "BURNING") {
        if (a || wasLogoUp) {
            wasLogoUp = false
            return "SKY"
        }
    } else if (state == "GAMEOVER" || state == "WAITEJECT") {
        if (b || wasScreenDown) {
            wasScreenDown = false
            return "UPSIDEDOWN"
        }
    }
    return "NONE"
}

/**
 * start timer by sampling runningtime and storing into starttime
 */
function startTimer() {
    timer = input.runningTime()
}

/**
 * get the elapsed time from the global starttime with ref to running time
 * in seconds.
 */
function getTimerSec(): number {
    let t = (input.runningTime() - timer) / 1000
    return t
}

/**
 * Show a splash screen "Perfect Pancakes >>>"
 * Splash screen "PP" with little arrow pointing to the start button
 */
function splashScreen() {
    let splash = images.createImage(`
        # # # # .   . . . . .   # # # # .   . . . . .   . . . . .   . . . . .   . . . . .
        # . . . #   . . . . .   # . . . #   # . . . .   . # . . .   . . # . .   . . . # .
        # # # # .   . . . . .   # # # # .   . # . . .   . . # . .   . . . # .   . . . . #
        # . . . .   . . . . .   # . . . .   # . . . .   . # . . .   . . # . .   . . . # .
        # . . . .   . . . . .   # . . . .   . . . . .   . . . . .   . . . . .   . . . . .
        `)
    // use show image (not show animation) so that button press is more responsive
    let index = 0
    // Any button press finishes the splash screen
    while (!input.buttonIsPressed(Button.B) && !input.buttonIsPressed(Button.A)) {
        splash.showImage(index * 5)
        index = index + 1
        if (index > splash.width() / 5) {
            index = 0
        }
        basic.pause(250)
    }
}

function testShake() { }

```

```blocks
let sugarThreshold: number
let ketoneThreshold: number

// Important parameters
sugarThreshold = 14
ketoneThreshold = 6
while (true) {
    // splash screen sugar cube (just the right sugar)
    basic.showAnimation(`
        # # # . .   . . . . .   . . . . .   . . . . .
        # # # . .   . # # # .   . . . . .   . . . . .
        # # # . .   . # # # .   . . # # #   . . # # #
        . . . . .   . # # # .   . . # # #   . . # # #
        . . . . .   . . . . .   . . # # #   . . # # #
        `, 400)
    // ask questions and give advice
    quiz()
    // wait for button press before restart
    waitAnyButton()
}

function getSugar(): string {
    waitNoButtons()
    let selection = "MID"
    while (!input.buttonIsPressed(Button.B)) {
        // high, low, normal?
        basic.showAnimation(`
            . . . . .   # . # . .   . . # . .
            . # # # .   # . # . .   . # . # .
            . # # # .   # . # # #   . . . # .
            . # # # .   # . . # .   . . # . .
            . . . . .   # . . # .   . . # . .
            `, 400)
        // show low, mid, or high as a bar
        selection = getLowMidHigh(selection)
    }
    return selection
}

function getKetone(): string {
    waitNoButtons()
    let selection = "MID"
    while (!input.buttonIsPressed(Button.B)) {
        // high, low, normal?
        basic.showAnimation(`
            . . . . .   . . . # .   . . # . .
            # # . . .   . . # . .   . # . # .
            # # # # #   . . # # .   . . . # .
            # # . . #   . . # . #   . . # . .
            . . . . .   # . . # .   . . # . .
            `, 400)
        // show low, mid, or high as a bar
        selection = getLowMidHigh(selection)
    }
    return selection
}

function getIll(): boolean {
    waitNoButtons()
    let selection = false
    while (!input.buttonIsPressed(Button.B)) {
        // ask question 'happy or sad'
        basic.showAnimation(`
            . . . . .   . . . . .   . . # . .   . . # . .
            . # . # .   . # . # .   . # . # .   . # . # .
            . . . . .   . . . . .   . . . # .   . . . # .
            # . . . #   . # # # .   . . # . .   . . # . .
            . # # # .   # . . . #   . . # . .   . . # . .
            `, 400)
        // get answer from user
        selection = getHappySad(selection)
    }
    // if we are happy, we are not ill
    return !selection
}

function getLowMidHigh(selection: string): string {
    let timeout = 2000
    let timer = input.runningTime()
    while (true) {
        // show the present level as a line
        if (selection == "LOW") {
            basic.showLeds(`
                . . . . .
                . . . . .
                . . . . .
                . . . . .
                # # # # #
                `)
        } else if (selection == "HIGH") {
            basic.showLeds(`
                # # # # #
                . . . . .
                . . . . .
                . . . . .
                . . . . .
                `)
        } else {
            basic.showLeds(`
                . . . . .
                . . . . .
                # # # # #
                . . . . .
                . . . . .
                `)
        }
        // process any user input
        if (input.buttonIsPressed(Button.A)) {
            // cycle round the 3 possible levels
            if (selection == "LOW") {
                selection = "MID"
            } else if (selection == "MID") {
                selection = "HIGH"
            } else {
                selection = "LOW"
            }
            // This is the 'hold repeat' time if you hold the A button
            basic.pause(300)
            // restart timer, 2 seconds of inactivity before return to main
            timer = input.runningTime()
        } else if (input.buttonIsPressed(Button.B)) {
            // user is selecting so better return quickly
            return selection
        } else if (input.runningTime() > timer + timeout) {
            // timeout, so return to main to re-prompt user
            return selection
        } else {
            // This slows the loop down to stop the emulator being busy
            // and also preserves battery life on the micro:bit
            // it also affects the worst case response time
            // it also affects the response time
            basic.pause(100)
        }
    }
}

/**
 * Wait for all buttons to be released
 * This prevents spurious selection in next question
 */
function waitNoButtons() {
    while (input.buttonIsPressed(Button.A) || input.buttonIsPressed(Button.B)) {
        basic.pause(100)
    }
}

function getHappySad(selection: boolean): boolean {
    let timeout = 2000
    let timer = input.runningTime()
    while (!input.buttonIsPressed(Button.B)) {
        // Show present selection
        if (selection) {
            basic.showLeds(`
                . . . . .
                . # . # .
                . . . . .
                # . . . #
                . # # # .
                `)
        } else {
            basic.showLeds(`
                . . . . .
                . # . # .
                . . . . .
                . # # # .
                # . . . #
                `)
        }
        // Wait for any change in the selection
        if (input.buttonIsPressed(Button.A)) {
            selection = !selection
            // This is the repeat time if button A held down
            basic.pause(500)
            timer = input.runningTime()
        } else if (input.runningTime() > timer + timeout) {
            return selection
        } else {
            // Preserve battery life on micro:bit and prevent emulator being busy
            // This is also the response time to a button press
            basic.pause(100)
        }
    }
    return selection
}

/**
 * Button A changes value, Button B selects value
 */
function quiz() {
    let sugar = getSugar()
    if (sugar != "HIGH") {
        // All is ok (tick)
        basic.showAnimation(`
            . . . . .   . . . . .   . . . . .   . . . . .   . . . . .
            . . . . .   . . . . .   . . . . .   . . . . .   . . . . #
            . . . . .   . . . . .   . . . . .   . . . # .   . . . # .
            # . . . .   # . . . .   # . # . .   # . # . .   # . # . .
            . . . . .   . # . . .   . # . . .   . # . . .   . # . . .
            `, 400)
    } else {
        // Button A changes value, Button B selects value
        let ketone = getKetone()
        if (ketone != "HIGH") {
            // Button A changes value, Button B selects value
            let ill = getIll()
            if (!ill) {
                // Time to rest (jump into bed)
                basic.showAnimation(`
                    . . . . .   # # . . .   . # . . .   . # . . .   . . . # .   . . . . .
                    . . . . #   . . . . #   # # . . #   . # . . #   . . # . #   . . . . #
                    . . . . #   . . . . #   . . . . #   # # . . #   # # . . #   # # # # #
                    # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
                    # . . . #   # . . . #   # . . . #   # . . . #   # . . . #   # . . . #
                    `, 400)
            } else {
                // Test more often (clock shows 2 hour interval)
                basic.showAnimation(`
                    . . # . .   . . . . .   . . . . .   . . . . .   . # # . .   . . . . .   . # # . .   . . . . .   . # # . .
                    . . # . .   . . . . .   . . . . .   . . . . .   . . . # .   . . . . .   . . . # .   . . . . .   . . . # .
                    . . # . .   . . # # #   . . # . .   # # # . .   . . # . .   . . . . .   . . # . .   . . . . .   . . # . .
                    . . . . .   . . . . .   . . # . .   . . . . .   . # . . .   . . . . .   . # . . .   . . . . .   . # . . .
                    . . . . .   . . . . .   . . # . .   . . . . .   . # # # .   . . . . .   . # # # .   . . . . .   . # # # .
                    `, 400)
            }
        } else {
            // Get some help (call the diabetes care team on the phone)
            basic.showAnimation(`
                . . . . .   . . . . .   # # # # #   # # . . .   # # . . .   # # . . .   # # . . .   # # # . .   # # . # .   # # . . #
                . . . . .   # # # # #   # . . . #   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .
                # # # # #   # . . . #   . . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .
                # . # . #   . . # . .   . . # . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .   # . . . .
                . # # # .   . # # # .   . # # # .   # # . . .   # # . . #   # # . # .   # # # . .   # # . . .   # # . . .   # # . . .
                `, 400)
        }
    }
}

function waitAnyButton() {
    while (!input.buttonIsPressed(Button.A) && !input.buttonIsPressed(Button.B)) {
        basic.pause(100)
    }
}

```

```blocks
let AWasPressed: boolean
let BWasPressed: boolean
let wasShake: boolean
let dots025: Image

dots025 = images.createImage(`
    . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
    . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
    . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
    . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .   # # . . .   # # # . .   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
    . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #
    `)
let tests = "DABPXYZSCT"
let test = 0
let prevTest = -1
startIOMonitor()
while (true) {
    let testLetter = tests[test]
    let autoRun = false
    if (testLetter == "D" || testLetter == "A" || testLetter == "B") {
        autoRun = true
    }
    if (!(test == prevTest)) {
        basic.showString(tests[test], 200)
        prevTest = test
    }
    if (AWasPressed || autoRun) {
        AWasPressed = false
        if (testLetter == "D") {
            testDisplay()
            test = test + 1
        } else if (testLetter == "A") {
            testButtonA()
            test = test + 1
        } else if (testLetter == "B") {
            testButtonB()
            test = test + 1
        } else if (testLetter == "P") {
            testPads()
        } else if (testLetter == "X") {
            testTiltX()
        } else if (testLetter == "Y") {
            testTiltY()
        } else if (testLetter == "Z") {
            testTiltZ()
        } else if (testLetter == "S") {
            testShake()
        } else if (testLetter == "C") {
            testCompass()
        } else if (testLetter == "T") {
            testTemperature()
        } else {
            // end of tests
            basic.showLeds(`
                . . . . .
                . . . . #
                . . . # .
                # . # . .
                . # . . .
                `, 400)
        }
        prevTest = -1
        AWasPressed = false
        BWasPressed = false
    } else if (BWasPressed) {
        BWasPressed = false
        if (test < tests.length - 1) {
            test = test + 1
        } else {
            test = 3
        }
    } else {
        basic.pause(100)
    }
}

/**
 * flash all LEDs 5 times
 */
function testDisplay() {
    for (let i = 0; i < 5; i++) {
        basic.plotLeds(`
            # # # # #
            # # # # #
            # # # # #
            # # # # #
            # # # # #
            `)
        basic.pause(200)
        basic.clearScreen()
        basic.pause(200)
    }
    // cycle all LEDs from 1 to 25
    basic.showAnimation(`
        # . . . .   # # . . .   # # # . .   # # # # .   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
        . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
        . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
        . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #   # # # # #
        . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   . . . . .   # . . . .   # # . . .   # # # . .   # # # # .   # # # # #
        `, 400)
}

function testButtonA() {
    basic.plotLeds(`
        . . # . .
        . # . # .
        . # # # .
        . # . # .
        . # . # .
        `)
    // wait for A pressed
    while (!input.buttonIsPressed(Button.A)) {
        basic.pause(100)
    }
    basic.plotLeds(`
        . # . . .
        # . # . .
        # # # . .
        # . # . .
        # . # . .
        `)
    // wait for A released
    while (input.buttonIsPressed(Button.A)) {
        basic.pause(100)
    }
    basic.plotLeds(`
        . . # . .
        . # . # .
        . # # # .
        . # . # .
        . # . # .
        `)
    basic.pause(1000)
}

function testTiltX() {
    basic.clearScreen()
    let prevx = 0
    while (!AWasPressed && !BWasPressed) {
        basic.pause(100)
        let x = input.acceleration(Dimension.X)
        let x2 = x / 512 + 2
        let x3 = Math.clamp(0, 4, x2)
        // sticky trace
        led.plot(x3, 0)
        // middle line is actual/live
        if (x3 != prevx) {
            led.unplot(prevx, 2)
            prevx = x3
        }
        led.plot(x3, 2)
        // bottom line is -4G, -2G, 1G, +2G, +4G
        if (x <= -2048) {
            led.plot(0, 4)
        } else if (x <= -1024) {
            led.plot(1, 4)
        } else if (x <= 1024) {
            led.plot(2, 4)
        } else if (x <= 2048) {
            led.plot(3, 4)
        } else {
            led.plot(4, 4)
        }
    }
}

function testShake() {
    wasShake = false
    basic.plotLeds(`
        . . . . .
        . . . . .
        . . # . .
        . . . . .
        . . . . .
        `)
    while (!AWasPressed && !BWasPressed) {
        if (wasShake) {
            wasShake = false
            basic.plotLeds(`
                # # # # #
                # # # # #
                # # # # #
                # # # # #
                # # # # #
                `)
            basic.pause(500)
            basic.plotLeds(`
                . . . . .
                . . . . .
                . . # . .
                . . . . .
                . . . . .
                `)
        } else {
            basic.pause(100)
        }
    }
}

function testCompass() {
    if (input.compassHeading() < 0) {
        input.calibrate()
    }
    basic.clearScreen()
    while (!AWasPressed && !BWasPressed) {
        let d = input.compassHeading()
        d = d / 22
        d = Math.clamp(0, 15, d)
        d = (d + 2) % 16
        if (d < 4) {
            led.plot(d, 0)
        } else if (d < 8) {
            led.plot(4, d - 4)
        } else if (d < 12) {
            led.plot(4 - d - 8, 4)
        } else {
            led.plot(0, 4 - d - 12)
        }
        basic.pause(100)
    }
}

function testPads() {
    let TESTSPEED = 500
    AWasPressed = false
    BWasPressed = false
    // Make sure all pins are inputs, before test starts
    let p0 = pins.digitalReadPin(DigitalPin.P0)
    let p1 = pins.digitalReadPin(DigitalPin.P1)
    let p2 = pins.digitalReadPin(DigitalPin.P2)
    let ok0 = 0
    let ok1 = 0
    let ok2 = 0
    while (!AWasPressed && !BWasPressed) {
        basic.clearScreen()
        // ## P0 out low, read from P1 and P2
        ok0 = 0
        pins.digitalWritePin(DigitalPin.P0, 0)
        basic.pause(TESTSPEED)
        p1 = pins.digitalReadPin(DigitalPin.P1)
        p2 = pins.digitalReadPin(DigitalPin.P2)
        if (p1 == 0) {
            led.plot(0, 0)
            ok0 = ok0 + 1
        }
        if (p2 == 0) {
            led.plot(1, 0)
            ok0 = ok0 + 1
        }
        // ## P0 out high, read from P1 and P2
        pins.digitalWritePin(DigitalPin.P0, 1)
        basic.pause(TESTSPEED)
        p1 = pins.digitalReadPin(DigitalPin.P1)
        p2 = pins.digitalReadPin(DigitalPin.P2)
        if (p1 == 1) {
            led.plot(2, 0)
            ok0 = ok0 + 1
        }
        if (p2 == 1) {
            led.plot(3, 0)
            ok0 = ok0 + 1
        }
        // set back to an input
        p0 = pins.digitalReadPin(DigitalPin.P0)
        // ## P1 out low, read from P0 and P2
        ok1 = 0
        pins.digitalWritePin(DigitalPin.P1, 0)
        basic.pause(TESTSPEED)
        p0 = pins.digitalReadPin(DigitalPin.P0)
        p2 = pins.digitalReadPin(DigitalPin.P2)
        if (p0 == 0) {
            led.plot(0, 1)
            ok1 = ok1 + 1
        }
        if (p2 == 0) {
            led.plot(1, 1)
            ok1 = ok1 + 1
        }
        // ## P1 out high, read from P0 and P2
        pins.digitalWritePin(DigitalPin.P1, 1)
        basic.pause(TESTSPEED)
        p0 = pins.digitalReadPin(DigitalPin.P0)
        p2 = pins.digitalReadPin(DigitalPin.P2)
        if (p0 == 1) {
            led.plot(2, 1)
            ok1 = ok1 + 1
        }
        if (p2 == 1) {
            led.plot(3, 1)
            ok1 = ok1 + 1
        }
        // set back to an input
        p0 = pins.digitalReadPin(DigitalPin.P1)
        // ## P2 out low, read from P0 and P1
        ok2 = 0
        pins.digitalWritePin(DigitalPin.P2, 0)
        basic.pause(TESTSPEED)
        p0 = pins.digitalReadPin(DigitalPin.P0)
        p1 = pins.digitalReadPin(DigitalPin.P1)
        if (p0 == 0) {
            led.plot(0, 2)
            ok2 = ok2 + 1
        }
        if (p1 == 0) {
            led.plot(1, 2)
            ok2 = ok2 + 1
        }
        // ## P2 out high, read from P0 and P1
        pins.digitalWritePin(DigitalPin.P2, 1)
        basic.pause(TESTSPEED)
        p0 = pins.digitalReadPin(DigitalPin.P0)
        p1 = pins.digitalReadPin(DigitalPin.P1)
        if (p0 == 1) {
            led.plot(2, 2)
            ok2 = ok2 + 1
        }
        if (p1 == 1) {
            led.plot(3, 2)
            ok2 = ok2 + 1
        }
        p2 = pins.digitalReadPin(DigitalPin.P2)
        // ## Assess final test status
        if (ok0 == 4) {
            led.plot(4, 0)
        }
        basic.pause(TESTSPEED)
        if (ok1 == 4) {
            led.plot(4, 1)
        }
        basic.pause(TESTSPEED)
        if (ok2 == 4) {
            led.plot(4, 2)
        }
        basic.pause(TESTSPEED)
        if (ok0 + ok1 + ok2 == 12) {
            // all tests passed
            led.plot(4, 4)
        }
        // ## Test cycle finished
        basic.pause(1000)
    }
    // intentionally don't clear A and B flags, so main loop can process them.
}

/**
 * - show number of dots on screen (0..25) to represent temperature in celcius
 */
function testTemperature() {
    while (!AWasPressed && !BWasPressed) {
        let temp = input.temperature() - 10
        temp = Math.clamp(0, 25, temp)
        dots025.plotFrame(temp)
        basic.pause(500)
    }
}

function testButtonB() {
    basic.plotLeds(`
        . # # . .
        . # . # .
        . # # . .
        . # . # .
        . # # . .
        `)
    // wait for B pressed
    while (!input.buttonIsPressed(Button.B)) {
        basic.pause(100)
    }
    basic.plotLeds(`
        . . # # .
        . . # . #
        . . # # .
        . . # . #
        . . # # .
        `)
    // wait for B released
    while (input.buttonIsPressed(Button.B)) {
        basic.pause(100)
    }
    basic.plotLeds(`
        . # # . .
        . # . # .
        . # # . .
        . # . # .
        . # # . .
        `)
    basic.pause(1000)
}

function testTiltY() {
    basic.clearScreen()
    let prevy = 0
    while (!AWasPressed && !BWasPressed) {
        basic.pause(100)
        let y = input.acceleration(Dimension.Y)
        let y2 = y / 512 + 2
        let y3 = Math.clamp(0, 4, y2)
        // sticky trace
        led.plot(0, y3)
        // middle line is actual/live
        if (y3 != prevy) {
            led.unplot(2, prevy)
            prevy = y3
        }
        led.plot(2, y3)
        // bottom line is -4G, -2G, 1G, +2G, +4G
        if (y <= -2048) {
            led.plot(4, 0)
        } else if (y <= -1024) {
            led.plot(4, 1)
        } else if (y <= 1024) {
            led.plot(4, 2)
        } else if (y <= 2048) {
            led.plot(4, 3)
        } else {
            led.plot(4, 4)
        }
    }
}

function testTiltZ() {
    basic.clearScreen()
    while (!AWasPressed && !BWasPressed) {
        let z = input.acceleration(Dimension.Z)
        if (z < -2000) {
            basic.plotLeds(`
                # . . . #
                # . . . #
                # . . . #
                # . . . #
                # . . . #
                `)
        } else if (z <= -1030) {
            basic.plotLeds(`
                . # . # .
                . # . # .
                . # . # .
                . # . # .
                . # . # .
                `)
        } else if (z <= 1000) {
            basic.plotLeds(`
                . . # . .
                . . # . .
                . . # . .
                . . # . .
                . . # . .
                `)
        } else if (z <= 1030) {
            basic.plotLeds(`
                . . . . .
                . . . . .
                # # # # #
                . . . . .
                . . . . .
                `)
        } else if (z <= 2000) {
            basic.plotLeds(`
                . . . . .
                # # # # #
                . . . . .
                # # # # #
                . . . . .
                `)
        } else {
            basic.plotLeds(`
                # # # # #
                . . . . .
                . . . . .
                . . . . .
                # # # # #
                `)
        }
        basic.pause(100)
    }
}

function startIOMonitor() {
    input.onButtonPressed(Button.A, () => {
        AWasPressed = true
    })
    input.onButtonPressed(Button.B, () => {
        BWasPressed = true
    })
    input.onShake(() => {
        wasShake = true
    })
}

```
