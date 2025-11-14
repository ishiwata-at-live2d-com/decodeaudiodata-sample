<script lang="ts">
  import { onMount } from "svelte";

  let canvas: HTMLCanvasElement;

  const FILE_SELECTOR = [
    { name: "ffmpeg mp4", path: "/ffmpeg_tone12db.mp4" },
    { name: "ffmpeg mkv", path: "/ffmpeg_tone12db.mkv" },
    { name: "obs mp4", path: "/obs_tone12db.mp4" },
  ];

  let audioCtx = $state<AudioContext | null>(null);
  let audioBuffer = $state<AudioBuffer | null>(null);
  let isPlaying = $state(false);
  let currentTime = $state(0);
  let startTime = $state(0);
  let animationId = $state<number | null>(null);
  let currentSource = $state<AudioBufferSourceNode | null>(null);

  let selectedAudio: string = $state<string>("obs mp4");
  let selectedFile = $derived.by(() => {
    // Update selected file based on selected audio name
    const file = FILE_SELECTOR.find((f) => f.name === selectedAudio);
    return file ? file.path : FILE_SELECTOR[0].path;
  });

  onMount(() => {
    audioCtx = new AudioContext({ sampleRate: 48000 });

    // キャンバスの初期設定
    if (canvas) {
      canvas.width = 800;
      canvas.height = 200;
    }

    return () => {
      audioCtx?.close();
      audioCtx = null;
    };
  });

  function drawWaveform(buffer: AudioBuffer, playbackPosition = 0) {
    if (!canvas) return;

    const ctx = canvas.getContext("2d");
    if (!ctx) return;

    const width = canvas.width;
    const height = canvas.height;
    const channelData = buffer.getChannelData(0); // 左チャンネルのデータを取得

    // キャンバスをクリア
    ctx.clearRect(0, 0, width, height);

    // 波形のスタイル設定
    ctx.strokeStyle = "#00ff00";
    ctx.lineWidth = 1;

    // 中央線を描画
    ctx.beginPath();
    ctx.strokeStyle = "#666666";
    ctx.moveTo(0, height / 2);
    ctx.lineTo(width, height / 2);
    ctx.stroke();

    // 波形を描画
    ctx.beginPath();
    ctx.strokeStyle = "#00ff00";

    const step = Math.ceil(channelData.length / width);
    let x = 0;

    for (let i = 0; i < channelData.length; i += step) {
      const amplitude = channelData[i];
      const y = ((amplitude + 1) / 2) * height; // -1から1の範囲を0からheightに変換

      if (i === 0) {
        ctx.moveTo(x, y);
      } else {
        ctx.lineTo(x, y);
      }
      x++;
    }

    ctx.stroke();

    // 再生位置インジケーターを描画
    if (playbackPosition > 0 && buffer.duration > 0) {
      const progressX = (playbackPosition / buffer.duration) * width;
      ctx.beginPath();
      ctx.strokeStyle = "#ff0000";
      ctx.lineWidth = 2;
      ctx.moveTo(progressX, 0);
      ctx.lineTo(progressX, height);
      ctx.stroke();

      // 再生時間の表示
      ctx.fillStyle = "#ff0000";
      ctx.font = "12px Arial";
      const timeText = `${playbackPosition.toFixed(1)}s / ${buffer.duration.toFixed(1)}s`;
      ctx.fillText(timeText, 10, 20);
    }
  }

  function updatePlaybackPosition() {
    if (!isPlaying || !audioCtx || !audioBuffer) return;

    currentTime = audioCtx.currentTime - startTime;

    // 音声の終了を検出
    if (currentTime >= audioBuffer.duration) {
      stopPlayback();
      return;
    }

    drawWaveform(audioBuffer, currentTime);
    animationId = requestAnimationFrame(updatePlaybackPosition);
  }

  async function loadAudio() {
    if (!audioCtx) return;

    try {
      const response = await fetch(selectedFile);
      const arrayBuffer = await response.arrayBuffer();
      audioBuffer = await audioCtx.decodeAudioData(arrayBuffer);

      // 音声ファイルを読み込んだら波形を描画
      drawWaveform(audioBuffer);
    } catch (error) {
      console.error("Error loading audio:", error);
    }
  }

  function stopPlayback() {
    isPlaying = false;
    if (animationId) {
      cancelAnimationFrame(animationId);
      animationId = null;
    }
    if (currentSource) {
      try {
        currentSource.stop();
      } catch (e) {
        // Already stopped
      }
      currentSource = null;
    }
    currentTime = 0;

    // キャンバスを削除
    if (audioBuffer) {
      drawWaveform(audioBuffer);
    }
  }

  function playSound() {
    if (!audioCtx || !audioBuffer) return;

    if (isPlaying) {
      stopPlayback();
      return;
    }

    const source = audioCtx.createBufferSource();
    source.buffer = audioBuffer;
    source.connect(audioCtx.destination);

    // 再生終了時のイベントリスナーを追加
    source.onended = () => {
      stopPlayback();
    };

    currentSource = source;
    startTime = audioCtx.currentTime;
    isPlaying = true;

    source.start();
    updatePlaybackPosition();
  }
</script>

<select bind:value={selectedAudio}>
  {#each FILE_SELECTOR as file}
    <option value={file.name} selected={file.name === selectedAudio}>
      {file.name}
    </option>
  {/each}
</select>

<div>
  <button onclick={loadAudio}> Load Audio </button>
  <button onclick={playSound} disabled={!audioBuffer}>
    {isPlaying ? "STOP" : "PLAY"}
  </button>
  <button onclick={stopPlayback} disabled={!audioBuffer || !isPlaying}>
    RESET
  </button>
</div>

<canvas bind:this={canvas} style="border: 1px solid #ccc; margin: 20px 0;">
</canvas>

<video src={selectedFile} controls> </video>
