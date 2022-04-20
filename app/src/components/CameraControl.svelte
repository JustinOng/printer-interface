<script lang="ts">
  import { onMount, createEventDispatcher } from "svelte";
  import Card, { Content } from "@smui/card";
  import Select, { Option } from "@smui/select";
  import CircularProgress from "@smui/circular-progress";
  import Tooltip, { Wrapper } from "@smui/tooltip";
  import { Icon } from "@smui/common";

  const dispatch = createEventDispatcher();

  let loading: boolean = true;
  let permissionDenied: boolean = false;
  let videoInputs: MediaDeviceInfo[] = [];

  let videoDisplay;

  onMount(async () => {
    try {
      await navigator.mediaDevices.getUserMedia({ video: true });
    } catch (error) {
      if (error.toString().includes("Permission denied")) {
        error =
          "Camera access has to be granted for this application to capture images.";
        permissionDenied = true;
      }

      dispatch("error", { error });
    }
    const devices = await navigator.mediaDevices.enumerateDevices();
    videoInputs = devices.filter(
      (device) => device.label && device.kind === "videoinput"
    );
    loading = false;
  });

  function onVideoSourceSelect(evt) {
    const deviceId = evt.detail.value;

    if (videoDisplay && deviceId) {
      navigator.mediaDevices
        .getUserMedia({
          video: {
            deviceId: { exact: deviceId },
          },
        })
        .then((stream) => {
          videoDisplay.srcObject = stream;
        });
    } else {
      videoDisplay.srcObject = undefined;
    }
  }
</script>

<Card padded style="max-width: 500px">
  <h3 style="margin: 0">
    Video Control
    {#if loading}
      <CircularProgress style="height: 0.95em; width: 0.95em;" indeterminate />
    {/if}
    {#if permissionDenied}
      <Wrapper>
        <Icon class="material-icons" style="font-size: 1em;">warning</Icon>
        <Tooltip
          >Camera access has to be granted for this application to capture
          images. Please grant the permission and refresh the page.</Tooltip
        >
      </Wrapper>
    {/if}
  </h3>
  <Content
    ><Select
      on:MDCSelect:change={onVideoSourceSelect}
      label="Device"
      disabled={loading || permissionDenied}
    >
      {#each videoInputs as input}
        <Option value={input.deviceId}>{input.label}</Option>
      {/each}
      <svelte:fragment slot="helperText"
        >Usually USB2.0 UVC PC Camera</svelte:fragment
      >
    </Select>

    <hr />

    <video bind:this={videoDisplay} style="width: 100%" playsinline autoplay>
      <track kind="captions" />
    </video>
  </Content>
</Card>