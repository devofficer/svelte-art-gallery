<script>
  import Meta from "./Meta.svelte";
  import Window from "./helpers/Window.svelte";
  import Main from "./Main.svelte";
  import Footer from "./prose/Footer.svelte";
  import Ceased from "./prose/Ceased.svelte";

  import { desisted } from ".././stores/global.js";
</script>

<Meta />

{#if $desisted}
  <Ceased />
{:else}
  <Window />

  <Main />

  <Footer />
{/if}

<style lang="scss" global>
  html {
    scroll-behavior: smooth;
    touch-action: manipulation;
  }

  a[href] {
    text-decoration: none;

    &:hover {
      text-decoration: underline;
    }
  }
  .highlight-text {
    font-weight: 400;
    letter-spacing: -0.025rem;
    padding: 3px 5px;
    border-radius: 3px;
    box-decoration-break: clone;
    display: inline-block;
    background: steelblue;
    background: linear-gradient(to left, white 50%, steelblue 50%) right;
    background-size: 200% 1.2em;
    transition: 1s ease-out 200ms;
  }

  /* SCROLLAMA */
  .chart {
    border: none;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    margin-top: 30px;
  }

  .step {
    padding: 1rem;
    // min-width: 300px;
    width: 90%;
    margin: 30rem auto;
    background-color: whitesmoke;
    border-left: 5px solid #cecece;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: #cecece;
    text-align: center;
    line-height: 1.5;
    opacity: 0.85;
    transition: 500ms all ease;

    &.active {
      color: black;
      border-left: 5px solid black;
      box-shadow: 1px 1px 7px 1px rgba(0, 0, 0, 0.12);
      opacity: 1;
      pointer-events: auto;
      background: white;

      .highlight {
        color: white;
        background-position: left;
      }
    }
  }

  .scrollama-container {
    display: flex;
    flex-direction: row-reverse;
    margin-bottom: 500px;

    // -webkit-touch-callout: none;
    // -webkit-user-select: none;
    // -khtml-user-select: none;
    // -moz-user-select: none;
    // -ms-user-select: none;
    // user-select: none;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);

    .scrollama-graphic {
      flex: 2;
      height: 95%;
    }

    .scrollama-steps {
      flex: 1;
      z-index: 1000;
      pointer-events: none;
    }

    @media screen and (max-width: 968px) {
      flex-direction: column;
      // align-items: center;
      // justify-content: center;
    }

    &.last {
      margin-bottom: 0;
    }
  }

  .scrollama-graphic {
    position: -webkit-sticky;
    position: sticky;
    top: 0;
  }
</style>
