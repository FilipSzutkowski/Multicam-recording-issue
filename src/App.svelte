<script lang="ts">
  import { tick } from 'svelte';
  import Streams from './Streams.svelte';

  let mediaDevices: MediaDeviceInfo[] = [];
  let streams: MediaStream[] = [];

  async function getDevices() {
    try {
      await navigator.mediaDevices.getUserMedia({
        video: true,
        audio: true,
      });
    } catch (error) {
      console.error(error);
    }

    await findDevices();
  }

  async function addDeviceStream(id: string, label: string) {
    if (!id) return;

    const audioDevice = mediaDevices.find(
      (device) => device.label === label && device.kind === 'audioinput'
    );

    console.log(audioDevice);

    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: {
          deviceId: { exact: id },
        },
        audio: {
          deviceId: audioDevice?.deviceId
            ? { exact: audioDevice.deviceId }
            : undefined,
        },
      });

      // Log the previous streams
      streams.forEach((s) => {
        const audioTracks = s.getAudioTracks();
        audioTracks.forEach((t) =>
          console.log(
            `Audio track: ${t.label}, Enabled: ${t.enabled}, ReadyState: ${t.readyState}`
          )
        );
      });

      streams = [...streams, stream];
      await addStreamToVideo(stream);
    } catch (error) {
      console.error(error);
    }
  }

  async function addStreamToVideo(stream: MediaStream) {
    await tick(); // Template has to be rendered before we can get the video element
    const video = document.getElementById(stream.id) as HTMLVideoElement;

    if (video) {
      video.srcObject = stream;
    } else {
      console.error('Video element not found');
    }
  }

  async function findDevices() {
    try {
      const foundDevices = await navigator.mediaDevices.enumerateDevices();
      mediaDevices = [];
      foundDevices.forEach((device) => {
        // if (device.kind === 'videoinput') {
        mediaDevices = [...mediaDevices, device];
        // }
      });
    } catch (error) {
      console.error(error);
    }
  }

  // $: console.log(streams);
  // $: console.log(mediaDevices);
</script>

<main
  class="bg-slate-900 text-slate-50 h-full flex flex-col items-start p-3 overflow-auto"
>
  <p class="text-2xl font-bold">Media Devices</p>
  <div class="flex my-3 flex-wrap gap-2">
    {#each mediaDevices.filter((d) => d.kind === 'videoinput') as { deviceId, label }}
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <div
        class="flex justify-center items-center border border-slate-400 rounded-md pl-3"
      >
        <p class="text-sm">{label}</p>
        <button
          on:click={() => addDeviceStream(deviceId, label)}
          class="text-3xl text-green-400 border-l border-slate-400 ml-3 px-3"
          >+</button
        >
      </div>
    {:else}
      <button
        class="border my-3 rounded-md py-2 px-4 font-medium"
        on:click={getDevices}>Get devices</button
      >
    {/each}
  </div>

  <Streams {streams} />
</main>

<style>
</style>
