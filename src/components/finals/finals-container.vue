<template>
    <div class="box">
        <div class="header">Finaler
            <span class="badge badge-light">{{numberOfFinals}}</span>
        </div>
        <finals-chart :chart-data="dataSet" class="canvas-size"></finals-chart>
        <filter-container :graph-type="graphType" v-show="!sharedFiltering"></filter-container>
    </div>
</template>

<script>
    import FinalsChart from "./finals-chart";
    import palette from "google-palette";
    import jmespath from "jmespath";
    import FilterContainer from "../filter/filter-container";
    import {FINALS_CHART} from "../../store/graph-types";

    export default {
        components: {
            FilterContainer,
            FinalsChart,
        },
        data() {
            return {
                numberOfFinals: 0,
                graphType: FINALS_CHART
            }
        },
        computed: {
            sharedFiltering() {
                return this.$store.getters.sharedFilter;
            },
            dataSet() {
                const zeroBasedWinnerObj = {};
                const zeroBasedRunnersUpObj = {};
                const number = this.$store.getters.numberOfPlayers;
                for (let i=1; i<number+1; i++) {
                    zeroBasedWinnerObj[i] = 0;
                    zeroBasedRunnersUpObj[i] = 0;
                }
                const games = this.$store.getters.games(this.graphType);
                const winsById = jmespath.search(games, "[*].GamesPlayed[*][].WinnerPlayerId").groupIds();
                const winners = Object.assign(zeroBasedWinnerObj, winsById);
                const runUpById = jmespath.search(games, "[*].GamesPlayed[*][].SecondPlayerId").groupIds();
                const runnersUp = Object.assign(zeroBasedRunnersUpObj, runUpById);
                const labels = Object.keys(winners).map(id => this.$store.getters.playerName(id));
                const firstPlace = Object.keys(winners).map(id => winners[id]);
                const secondPlace = Object.keys(runnersUp).map(id => runnersUp[id]);
                const backgroundColors = palette(this.graphColorScheme, 2).map(v => "#"+ v).reverse();
                this.numberOfFinals = firstPlace.reduce((acc, cur) => {
                    return acc + cur;
                }, 0);
                return {
                    labels: labels,
                    datasets: [
                        {
                            label: '1. plads',
                            backgroundColor: backgroundColors[0],
                            data: firstPlace
                        },
                        {
                            label: '2. plads',
                            backgroundColor: backgroundColors[1],
                            data: secondPlace
                        }
                    ]
                }
            }
        }
    }
</script>

<style scoped>
    .box {
        margin: 5px;
        width: 400px;
        border: 1px solid cadetblue;
    }
    .canvas-size {
        height: 400px;
    }
    .header {
        background-color: cadetblue;
        font-family: "Arial", serif;
        color: darkslategray;
        font-variant: all-small-caps;
        font-weight: bold;
        vert-align: middle;
        text-align: center;
    }
</style>
