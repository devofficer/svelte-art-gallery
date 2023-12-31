<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import debounceFn from "lodash.debounce";
  import Icon from "./helpers/Icon.svelte";
  import GridHistogram from "./GridHistogram.svelte";
  import { windowWidth } from "../stores/global.js";

  export let data, height;

  const padding = { top: 5, right: 0, bottom: 30, left: 0 };

  $: width = null;
  $: activeStep = 0;
  $: lastResponse = null;

  let currWindowWidth = $windowWidth;
  const wildernessDayColors = [
    { color: "Titanium White", hex: "#FFFFFF", text_color: "#000000" },
    { color: "Cadmium Yellow", hex: "#FFEC00", text_color: "#000000" },
    { color: "Indian Yellow", hex: "#FFB800", text_color: "#000000" },
    { color: "Yellow Ochre", hex: "#C79B00", text_color: "#000000" },
    { color: "Sap Green", hex: "#0A3410", text_color: "#FFFFFF" },
    { color: "Phthalo Blue", hex: "#0C0040", text_color: "#FFFFFF" },
    { color: "Bright Red", hex: "#DB0000", text_color: "#FFFFFF" },
    { color: "Dark Sienna", hex: "#5F2E1F", text_color: "#FFFFFF" },
    { color: "Alizarin Crimson", hex: "#4E1500", text_color: "#FFFFFF" },
    { color: "Black Gesso", hex: "#000000", text_color: "#FFFFFF" },
    { color: "Van Dyke Brown", hex: "#221B15", text_color: "#FFFFFF" },
    { color: "Midnight Black", hex: "#000000", text_color: "#FFFFFF" },
  ];

  // SCROLL
  onMount(async () => {
    // instantiate the scrollama
    const scroller = scrollama();

    function handleStepEnter(response) {
      activeStep = response.index;
      if (response.index == 0) {
        grid();
      }
      if (response.index == 1) {
        histogram();
      }
      if (response.index == 2) {
        highlight("number", 12, "steelblue");
      }
      if (response.index == 3) {
        highlight("index", 403, "steelblue");
      }
      if (response.index == 4) {
        highlight("number", 1, "#664228");
      }
      lastResponse = response;
    }

    // setup the instance, pass callback functions
    scroller
      .setup({
        step: "#gridSection .step",
      })
      .onStepEnter((response) => handleStepEnter(response))
      .onStepExit((response) => {});

    function resizeWidth() {
      // If width changed, trigger last step again
      if (window.innerWidth != currWindowWidth) {
        if (lastResponse) {
          handleStepEnter(lastResponse);
        }
      }
    }

    window.addEventListener("resize", resizeWidth);
    window.addEventListener("resize", debounceFn(scroller.resize, 300));
  });

  // A painting can have between 1 and 15 (maxColors) colors
  // We want an array of 1-15 for our X axis
  //  Using the default data.map(d => d.num_colors).sort((a,b) => a - b) would result in missing numbers (2) to not appear
  const maxColors = d3.max(data, (d) => d.num_colors);
  const xDomain = Array(maxColors)
    .fill()
    .map((element, index) => index + 1);

  $: xScaleHist = d3
    .scaleBand()
    .domain(xDomain)
    .range([padding.left, width - padding.right]);

  $: yScaleHist = d3
    .scaleLinear()
    .domain(d3.extent(data, (d) => d.yPos))
    .range([height - padding.bottom, padding.top]);

  $: xScaleGrid = d3
    .scaleBand()
    .domain(data.map((d) => d.gridX))
    .range([padding.left, width - padding.right]);

  $: yScaleGrid = d3
    .scaleBand()
    .domain(data.map((d) => d.gridY))
    .range([0, height - padding.bottom - padding.top]);

  $: xTicks = null;
  $: yTicks = null;

  $: rectWidthGrid = (width / 19) * 0.95;
  $: rectHeightGrid = (height / 21) * 0.95;

  $: rectWidthHist = width / d3.max(data, (d) => d.num_colors);
  $: rectHeightHist =
    (height - padding.top - padding.bottom) / d3.max(data, (d) => d.yPos);

  let initialized = false;

  function grid() {
    xTicks = null;
    yTicks = null;

    if (initialized) {
      d3.selectAll(".gridRect")
        .data(data)
        .transition("grid")
        // .delay((d) => (d.gridY * d.gridX) / 10)
        .duration(1000)
        .attr("x", (d) => xScaleGrid(d.gridX))
        .attr("y", (d) => yScaleGrid(d.gridY))
        .attr("width", rectWidthGrid)
        .attr("height", rectHeightGrid)
        .attr("fill", "grey")
        .attr("stroke", "white")
        .attr("opacity", 1);
    } else {
      d3.selectAll(".gridRect")
        .data(data)
        .attr("x", (d) => -rectWidthGrid)
        .attr("y", (d) => yScaleGrid(d.gridY))
        .attr("width", rectWidthGrid)
        .attr("height", rectHeightGrid)
        .attr("fill", "grey")
        .attr("stroke", "white")
        .attr("opacity", 1)
        .transition("init")
        .delay((d) => d.gridX * d.gridY)
        .duration(1000)
        .attr("x", (d) => xScaleGrid(d.gridX));

      initialized = true;
    }
  }

  function histogram() {
    xTicks = xScaleHist.domain();
    yTicks = [0, 50, 100];

    d3.selectAll(".gridRect")
      .data(data)
      .transition("histogram")
      // .delay((d) => d.num_colors * 15)
      .duration(1000)
      .attr("x", (d) => xScaleHist(d.num_colors))
      .attr("y", (d) => yScaleHist(d.yPos))
      .attr("width", rectWidthHist)
      .attr("height", rectHeightHist)
      .attr("fill", "grey")
      .attr("stroke", "white")
      .attr("stroke-width", ".5px")
      .attr("stroke-dasharray", "100%")
      .attr("opacity", 1);
  }

  function highlight(type, num, color) {
    xTicks = xScaleHist.domain();
    yTicks = [0, 50, 100];

    d3.selectAll(".gridRect")
      .data(data)
      .transition("highlight")
      // .delay((d) => -d.yPos * 5)
      .duration(1000)
      .attr("x", (d) => xScaleHist(d.num_colors))
      .attr("y", (d) => yScaleHist(d.yPos))
      .attr("width", rectWidthHist)
      .attr("height", rectHeightHist)
      .attr(
        "fill",
        type == "number"
          ? (d) => (d.num_colors == num ? color : "#cecece")
          : (d) => (d.painting_index == num ? color : "#cecece")
      )
      .attr("stroke-dasharray", `0, ${rectWidthHist}, ${rectHeightHist}, 0`);
  }
</script>

<div class="scrollama-container">
  <div class="scrollama-graphic">
    <div
      class="chart"
      id="grid"
      bind:offsetWidth={width}
      style="height: {height}px"
    >
      <div class="gridTip" />
      <svg style="width: 100%; height: 100%;">
        <GridHistogram
          {height}
          {data}
          {padding}
          {xScaleHist}
          {yScaleHist}
          {xTicks}
          {yTicks}
        />
      </svg>
    </div>
  </div>

  <div class="scrollama-steps" id="gridSection">
    <div class="step" class:active={activeStep == 0} data-step="a">
      <p>
        This is every piece Bob Ross painted in <em>The Joy of Painting</em>.
        Each rectangle
        <svg width="20" height="15">
          <rect width="100%" height="100%" fill="grey" />
        </svg>
        represents a painting.
      </p>
      <p>
        There are 403 rectangles, organized chronologically. Go ahead and hover
        <Icon name="mouse-pointer" stroke="black" strokeWidth="1" />
        over a rectangle to see the painting it represents!
      </p>
    </div>
    <div class="step" class:active={activeStep == 1} data-step="b">
      <p>
        To get a feeling for Bob Ross' process, we can organize the pieces by
        the <strong>number of colors</strong> used to paint them. The height of a
        bar represents how many paintings used that number of colors.
      </p>
    </div>
    <div class="step" class:active={activeStep == 2} data-step="c">
      <p>
        Most commonly, paintings have <span
          class="highlight-text"
          style="background: steelblue; color: white;">12 colors</span
        >. Of the {data.length} pieces in <em>The Joy of Painting</em>, {data.filter(
          (d) => d.num_colors == 12
        ).length} used 12 colors.
      </p>
      <p>
        The peak is concentrated around 12, meaning most of Ross' paintings used
        somewhere in the range of 7-13 colors; very rarely did they venture
        outside of that range.
      </p>
    </div>
    <div class="step" class:active={activeStep == 3} data-step="c">
      <p>
        A great example of this prototypical painting is <strong
          >Wilderness Day</strong
        >. This was <em>The Joy of Painting's</em> series finale, and Bob Ross' final
        painting before he passed away on July 4, 1995.
      </p>
      <a
        href="https://www.youtube.com/embed/nJGCVFn57U8"
        target="_blank"
        rel="noopener noreferrer"
        class="frame margin-vertical"
        ><img
          src="https://www.twoinchbrush.com/images/painting411.png"
          class="painting-in-text"
          alt="Wilderness Day by Bob Ross"
        />
      </a>
      <div class="colors" style="margin: 1rem 0">
        {#each wildernessDayColors as { color, hex, text_color }}
          <span
            class="color-label"
            style="background: {hex}; color: {text_color}">{color}</span
          >
        {/each}
      </div>
    </div>
    <div class="step" class:active={activeStep == 4} data-step="d">
      <p>
        On the contrary, one painting used <strong>only one color</strong>. That
        painting was <strong>Contemplative Lady</strong>, which was painted by a
        guest on episode 6 of season 16.
      </p>
      <a
        href="https://www.youtube.com/embed/gnp6WE7Ql-s"
        target="_blank"
        rel="noopener noreferrer"
        class="frame"
        ><img
          src="https://www.twoinchbrush.com/images/painting89.png"
          class="painting-in-text"
          alt="Contemplative Lady by Bob Ross"
        />
      </a>
      <p>
        The guest painted Contemplative Lady with one color: <span
          class="highlight-text"
          style="background: #664228; color: white;">Van Dyke Brown</span
        >.
      </p>
    </div>
  </div>
</div>

<style lang="scss">
  .gridTip {
    opacity: 0;
    position: absolute;
    background: white;
    border: 0px;
    border-radius: 3px;
    pointer-events: none;
    width: 200px;
  }

  .color-label {
    padding: 5px;
    margin: 2px;
    box-shadow: 1px 1px 6px 1px #cecece;
    border-radius: 5px;
  }
</style>
