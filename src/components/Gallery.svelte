<script>
  import GalleryPainting from "./GalleryPainting.svelte";
  import GalleryInformation from "./GalleryInformation.svelte";
  import Scrubber from "./Scrubber.svelte";
  import { painting_index } from "../stores/global.js";

  export let data;
  export let long_data;

  let selected = [];

  // How many art pieces there are in the dataset
  const LENGTH = data.length - 1;

  $: {
    selected = data[$painting_index] ? data[$painting_index] : data[0];
  }
</script>

<div class="gallery" id="gallery">
  <div class="background">
    <div class="gallery-bg">
      <button
        on:click={() => {
          painting_index.update((n) => (n - 1 < 0 ? 0 : n - 1));
        }}
        class={$painting_index === 0 ? "left disabled" : "left"}
      >
        &#8249;
      </button>
      <img
        src="./assets/background.png"
        alt="An empty art gallery wall"
        width="100%"
      />
      <div class="painting">
        <a
          href={selected.youtube_src}
          target="_blank"
          rel="noopener noreferrer"
        >
          <GalleryPainting {selected} />
        </a>
      </div>
      <button
        on:click={() => {
          painting_index.update((n) => (n + 1 > LENGTH ? LENGTH : n + 1));
        }}
        class={$painting_index === LENGTH ? "disabled right" : "right"}
      >
        &#8250;
      </button>
    </div>
    <Scrubber data={long_data} />
  </div>
  <div class="information">
    <GalleryInformation {selected} />
  </div>
</div>

<style lang="scss">
  button {
    background: white;
    box-shadow: 1px 1px 3px 1px #cecece;
    width: 2rem;
    height: 2rem;
    padding: 0;
    border-radius: 50%;

    &.disabled {
      cursor: not-allowed;
      background: #e4e4e4;
      color: #a8a8a8;
      box-shadow: none;
    }

    &:hover {
      background: whitesmoke;
      box-shadow: 0 0 3px 0 #cecece;
    }
  }
  .gallery {
    display: flex;
    // min-height: 80vh;

    > .background {
      flex: 2;
      background: white;
      display: flex;
      flex-direction: column;

      @media screen and (max-width: 968px) {
        flex-direction: column-reverse;
        background: whitesmoke;
      }

      .gallery-bg {
        position: relative;

        > button {
          position: absolute;
          top: 50%;

          &.left {
            left: 10px;
          }

          &.right {
            right: 10px;
          }
        }

        .painting {
          position: absolute;
          top: 30%;
          left: 50%;
          transform: translate(-50%, -50%);
          width: 33%;
          transition: 1s all ease;

          &:hover {
            transform: translate(-48%, -48%);
          }
        }
      }
    }

    > .information {
      flex: 1;
      display: flex;
      place-items: center;
      background: white;
    }
  }

  @media screen and (max-width: 968px) {
    .gallery {
      flex-direction: column;
    }
  }
</style>
