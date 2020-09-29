<script>
import { Bar } from "vue-chartjs";

export default {
  extends: Bar,
  props: {
    datasets: {
      type: Array,
    },
    labels: {
      type: Array,
    },
    options: {
        type: Object
    }
  },
  mounted() {
    this.renderChart(
      {
        labels: this.labels,
        datasets: this.datasets,
      },
      this.options
    );
  },
};
</script>