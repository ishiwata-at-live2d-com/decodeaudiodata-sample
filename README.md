# Sample Demonstrating Audio Loading Issues with Audio Context

## Overview

This is a sample that demonstrates noise issues when loading audio data using the `decodeAudioData` method of the audio context with specific MP4 files.
Note that we have confirmed that this issue does not occur when playing with the Video tag.

## How to Run

1. Clone the repository.
2. Install dependencies:
   ```bash
   npm install
   // or
   pnpm install
   ```
3. Start the development server:
   ```bash
   npm run dev
   // or
   pnpm dev
   ```
4. Access `http://localhost:5173` in your browser.
5. Select "obs mp4" from the dropdown menu and click the "Load Audio" button to load the audio.
6. Click the "Play Audio" button to play the audio. You should notice noise occurring. (You should also see noise in the waveform.)
   Also, confirm that no noise occurs when playing the same file with the Video tag.
7. Select "ffmpeg mp4" and follow the same procedure to confirm that no noise occurs.

## Notes

- This sample is intended to demonstrate that the issue occurs only with specific MP4 files. Not all MP4 files exhibit this problem.
- We have confirmed that similar issues occur in Chromium-based browsers (latest stable versions of Google Chrome and Microsoft Edge). The issue does not occur in Firefox.

## Additional Information

- This sample project is built using [SvelteKit](https://kit.svelte.dev/).
