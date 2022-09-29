<script>
  import { getContext } from "svelte"
  import Button from "../node_modules/@budibase/bbui/src/Button/Button.svelte"

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  const STAGES = {
    IDLE: "IDLE",
    RECORDING: "RECORDING",
    DONE: "DONE"
  }

  const EXTENSIONS = {
    "audio/wav": "wav",
    "audio/ogg": "ogg",
    "audio/mpeg": "mp3",
  }

  export let type = "audio/wav"
  export let filename = "recording"
  export let size
  export let disabled

  let stage = STAGES.IDLE
  let download
  let recorder

  async function handleRecord() {
    switch (stage) {
      case STAGES.IDLE:
        const recordedChunks = [];
        const stream = await navigator.mediaDevices.getUserMedia({ audio: true })

        recorder = new MediaRecorder(stream);

        recorder.addEventListener("dataavailable", function(evt) {
          if (evt.data.size > 0) { 
            recordedChunks.push(evt.data)
          }
        })

        recorder.addEventListener("stop", function() {
          download = {
            link: URL.createObjectURL(new Blob(recordedChunks)),
            filename
          }
        })

        recorder.start();
        stage = STAGES.RECORDING
        break
      case STAGES.RECORDING:
        recorder.stop();
        stage = STAGES.DONE
        break
      case STAGES.DONE:
        stage = STAGES.IDLE
        break
    }
  }
</script>

<div class="container" use:styleable={$component.styles}>
  <Button {size} {disabled} icon="VoiceOver" cta on:click={handleRecord}>
    {#if stage === STAGES.IDLE}
      Record
    {:else if stage === STAGES.RECORDING}
      Stop
    {:else if stage === STAGES.DONE}
      Reset
    {/if}
  </Button>
  {#if stage === STAGES.RECORDING}
    <div class="record-icon recording" />
  {/if}

  {#if stage === STAGES.DONE}
    <audio controls class="audio-player" {type} title={`${filename}.${EXTENSIONS[type]}`} src={download?.link} />
  {/if}
</div>


<style>
    .container {
      display: flex;
      align-items: center;
    }

    .record-icon {
      margin-left: 10px;
      height: 12px;
      width: 12px;
      border-radius: 100%;
      background: rgb(211, 21, 16);
    }

   .record-icon.recording {
        animation: pulse 2s infinite;
    }

    @keyframes pulse {
        0% {
            transform: scale(0.95);
            box-shadow: 0 0 0 0 rgba(211, 21, 16, 0.7);
        }

        70% {
            transform: scale(1);
            box-shadow: 0 0 0 10px rgba(211, 21, 16, 0);
        }

        100% {
            transform: scale(0.95);
            box-shadow: 0 0 0 0 rgba(211, 21, 16, 0);
        }
    }
</style>