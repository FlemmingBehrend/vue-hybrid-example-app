<script>
    import {Bar, mixins} from "vue-chartjs";

    export default {
        extends: Bar,
        mixins: [mixins.reactiveProp],
        props: {
            chartData: {
                type: Array | Object,
                required: true
            }
        },
        data() {
            return {
                options: {
                    scales: {
                        xAxes: [{
                            stacked: true
                        }],
                        yAxes: [{
                            stacked: true
                        }]
                    },
                    responsive: true,
                }
            };
        },
        mounted() {
            this.renderChart(
                this.chartData, this.options);
        }
    };
</script>
