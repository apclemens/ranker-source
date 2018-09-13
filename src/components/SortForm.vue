<template>
    <div>
        <progress v-bind:value="pairsDone" v-bind:max="items.length * (items.length-1) / 2"></progress><br>
        {{pairsDone * 2 / (items.length * (items.length-1))*100}}%<br>
        {{pairsDone}} / {{(items.length * (items.length-1) / 2)}}

        <div v-if="!done">
            <div v-if="!disableSkip" class="compare" id="dontCare" v-on:click="deferRun([itemA, itemB])">
                Don't care
            </div>
        <div class="compare" id="itemA" v-on:click="addRun([itemA, itemB])">
            {{items[itemA]}}
        </div>
        <div class="compare" id="itemB" v-on:click="addRun([itemB, itemA])">
            {{items[itemB]}}
        </div>
        </div><br>
        {{pairs}}<br>
        {{order}}<br>
        {{runs}}<br>
        <div v-if="done">
            <ul>
                <li v-for="(o, index) in order" v-bind:key="index">
                    {{items[o]}}
                </li>
            </ul>
        </div>
    </div>
</template>

<script>

export default {
    name: 'SortForm',
    props: ['items', 'sortMethod'],
    data() {
        return {
            itemA: '',
            itemB: '',
            pairs: [],
            order: [],
            runs: [],
            done: false,
            pairsDone: 0,
            lastSkipped: [-1, -1],
            disableSkip: false,
        }
    },
    mounted() {
        for (var i=0; i<this.items.length; i++) {
            for (var j=i+1; j<this.items.length; j++) {
                this.pairs.push([i, j])
            }
        }

        var ths = this;

        document.addEventListener('keypress', function(event) {
            if (event.key == 'ArrowLeft') {
                ths.addRun([ths.itemA, ths.itemB]);
            } else if (event.key == 'ArrowRight') {
                ths.addRun([ths.itemB, ths.itemA]);
            } else if (event.key == 'ArrowUp') {
                ths.deferRun([ths.itemA, ths.itemB]);
            }
        })

        this.pairs = shuffle(this.pairs);
        this.order = [...Array(this.items.length).keys()];
        this.askPreference();
    },
    methods: {
        askPreference: function() {
            if (this.pairs.length == 0) {
                while (this.order.length < this.items.length) {
                    for (var i=0; i<this.runs.length; i++) {
                        if (this.runs[i][0] == this.order[this.order.length-1]) {
                            this.order.push(this.runs[i][1]);
                        }
                    }
                }
                this.done = true;
                return;
            }
            var currentPair = shuffle(this.pairs.pop());
            if ((currentPair[0] == this.lastSkipped[0] && currentPair[1] == this.lastSkipped[1]) ||
                (currentPair[0] == this.lastSkipped[1] && currentPair[1] == this.lastSkipped[0])) {
                this.disableSkip = true;
            }
            this.pairsDone ++;
            if (trailPairs(currentPair[0], currentPair[1], this.runs)) return this.askPreference();
            if (trailPairs(currentPair[1], currentPair[0], this.runs)) return this.askPreference();

            this.itemA = currentPair[0];
            this.itemB = currentPair[1];
        },
        addRun: function(newRun) {
            var index=this.order.indexOf(newRun[1]);
            if (index >= 0) {
                this.order.splice(index, 1)
            }
            var toRemove = []
            for (var i=0; i<this.runs.length; i++) {
                if (trailPairs(this.runs[i][0], newRun[0], this.runs) &&
                    trailPairs(newRun[1], this.runs[i][1], this.runs)) {
                        toRemove.push(i)
                }
            }
            for (var j=toRemove.length-1; j>=0; j--) {
                this.runs.splice(toRemove[j],1);
            }
            this.runs.push(newRun)
            this.lastSkipped = [-1, -1];
            this.disableSkip = false;
            this.askPreference();
        },
        deferRun: function(run) {
            this.pairs.unshift(run);
            this.pairsDone --;
            if (this.lastSkipped[0] == -1) {
                this.lastSkipped[0] = run[0];
                this.lastSkipped[1] = run[1];
            }
            this.askPreference();
        },
    },
}

function shuffle(array) {
  var currentIndex = array.length, temporaryValue, randomIndex;

  // While there remain elements to shuffle...
  while (0 !== currentIndex) {

    // Pick a remaining element...
    randomIndex = Math.floor(Math.random() * currentIndex);
    currentIndex -= 1;

    // And swap it with the current element.
    temporaryValue = array[currentIndex];
    array[currentIndex] = array[randomIndex];
    array[randomIndex] = temporaryValue;
  }

  return array;
}

function trailPairs(start, end, runs) {
    if (start == end) return true;
    for (var i=0; i<runs.length; i++) {
        var run = runs[i];
        if (run[0] == start) {
            if (trailPairs(run[1], end, runs)) {
                return true;
            }
        }
    }
    return false;
}
</script>

<style>
.compare {
	cursor: pointer;
	width: 500px;
	height: 500px;
	border: 2px solid black;
	display: inline-block;
}
.remove {
    width: 50px;
    display: block;
}
#dontCare {
    display: block;
    width: 1004px;
    height: 50px;
}
</style>
