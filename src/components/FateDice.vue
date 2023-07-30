<template>
    <div class="main">
        <div class="app-container">
            <h1 class="app-title">Fate Dice Roller</h1>
            <div class="dice-container">
                <div class="dice" v-for="(result, index) in results" :key="'result-' + index">
                    {{ display[result] }}
                </div>
            </div>
            <div class="result-container">
                <h2 class="result-title">Roll Result:</h2>
                <div class="result-display">
                    <span class="result-sum" v-if="resultSum !== null">
                        {{ resultSum > 0 ? '+' : '' }}{{ resultSum }}
                    </span>
                    <span class="result-scaled" v-if="scaledResult !== null">
                        {{ scaledResult }}
                    </span>
                </div>
            </div>
            <button class="roll-button" @click="doRoll" :disabled="isRolling">
                {{ isRolling ? 'Rolling...' : 'Roll the Dice' }}
            </button>
            <div class="history-container">
                <h2 class="history-title">Roll History:</h2>
                <div class="scrollable-area">
                    <div class="latest-results" v-for="(result, index) in latest.slice(0, 5)" :key="'latest-' + index">
                        <div class="latest-item">
                            <span class="result-sum">{{ result.sum > 0 ? '+' : '' }}{{ result.sum }}</span>
                            <span class="result-scaled">{{ result.scaled }}</span>
                            <span class="roll-time">{{ result.time }}</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
  
<script>
export default {
    data() {
        return {
            latest: JSON.parse(localStorage.getItem('history')) || [],
            latestCount: 10,
            rollDelay: 100,
            rolls: 5,
            display: {
                '-1': '-',
                '0': '0',
                '1': '+'
            },
            scales: {
                'en': {
                    '-2': 'terrible',
                    '-1': 'poor',
                    '0': 'mediocre',
                    '1': 'average',
                    '2': 'fair',
                    '3': 'good',
                    '4': 'great',
                    '5': 'superb',
                    '6': 'fantastic',
                    '7': 'epic',
                    '8': 'legendary'
                },
                'it': {
                    '-2': 'pessimo',
                    '-1': 'scarso',
                    '0': 'mediocre',
                    '1': 'medio',
                    '2': 'discreto',
                    '3': 'buono',
                    '4': 'ottimo',
                    '5': 'eccellente',
                    '6': 'fantastico',
                    '7': 'epico',
                    '8': 'leggendario'
                }
            },
            lang: navigator.language,
            isRolling: false,
            results: [],
            resultSum: null,
            scaledResult: null,
        }
    },
    computed: {
        scale() {
            return this.scales[this.lang] || this.scales[this.lang.split('-')[0]] || this.scales['en'];
        },
    },
    methods: {
        getRandomInt(min, max) {
            return Math.floor(Math.random() * (max - min + 1)) + min;
        },
        rollDie() {
            return this.getRandomInt(-1, 1);
        },
        rollDice() {
            var results = [];
            for (var i = 0; i < 4; i++) {
                results[i] = this.rollDie();
            }
            return results;
        },
        sumDice(results) {
            return results.reduce(function (sum, d) { return sum + d }, 0);
        },
        doRoll() {
            if (this.isRolling)
                return;
            this.isRolling = true;
            this.displayBlankResult();
            this.rolls = 5;
            this.internalRoll();
        },
        internalRoll() {
            this.results = this.rollDice();
            this.resultSum = this.sumDice(this.results);
            this.scaledResult = this.scale[this.resultSum];
            if (--this.rolls) {
                setTimeout(this.internalRoll, this.rollDelay);
            } else {
                this.isRolling = false;
                this.saveResult(this.results);
            }
        },
        saveResult(results) {
            const time = new Date().toLocaleString();
            this.latest.unshift({
                results: results.slice(),
                sum: this.resultSum,
                scaled: this.scaledResult,
                time: time
            });
            while (this.latest.length > this.latestCount)
                this.latest.pop();
            localStorage.setItem('history', JSON.stringify(this.latest));
        },
        displayBlankResult() {
            this.resultSum = null;
            this.scaledResult = null;
        },
    },
    mounted() {
        this.doRoll();
    }
}
</script>

<style scoped>
body {
    background-color: #3E2723;
}

.main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    font-family: 'IM Fell English SC', serif;
}

.app-container {
    background-color: #FFECB3;
    max-width: 800px;
    width: 90%;
    padding: 40px;
    border: 2px solid #5D4037;
    border-radius: 12px;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
    text-align: center;
}

.app-title {
    font-size: 2.8em;
    color: #5D4037;
    margin-bottom: 20px;
}

.dice-container {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 30px 0;
    gap: 20px;
    flex-wrap: wrap;
}

.dice {
    border: 2px solid #5D4037;
    width: 60px;
    height: 60px;
    line-height: 60px;
    display: inline-flex;
    justify-content: center;
    align-items: center;
    font-size: 24px;
    color: #5D4037;
    background-color: #CFD8DC;
    border-radius: 12px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.result-container,
.history-container {
    margin: 30px 0;
    text-align: left;
}

.result-title,
.history-title {
    font-size: 1.6em;
    color: #5D4037;
}

.result-display {
    background: #CFD8DC;
    border-radius: 12px;
    padding: 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #5D4037;
    font-size: 2em;
}

.roll-button {
    color: #FFFFFF;
    background-color: #795548;
    border: none;
    padding: 16px 32px;
    font-size: 18px;
    border-radius: 8px;
    cursor: pointer;
    transition: background-color 0.2s ease;
    margin: 30px auto;
    display: block;
    width: 50%;
    max-width: 200px;
}

.roll-button:hover {
    background-color: #5D4037;
}

.roll-button:disabled {
    background-color: #B0BEC5;
    cursor: not-allowed;
}

.history-container {
    max-height: 200px;
    overflow-y: auto;
    margin: 30px 0;
    text-align: left;
    border: 2px solid #5D4037;
    border-radius: 12px;
    background-color: rgba(255, 255, 255, 0.6);
}

.latest-results {
    background-color: #ECEFF1;
    border-radius: 12px;
    padding: 20px;
    margin-top: 20px;
}

.latest-item {
    border-bottom: 1px solid #CFD8DC;
    padding: 10px 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.latest-item:last-child {
    border-bottom: none;
}
</style>