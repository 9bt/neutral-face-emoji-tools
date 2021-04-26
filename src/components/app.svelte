<script>
  import uuid from 'uuid';

  import Upload from './upload.svelte';
  import FileDropzone from './file-dropzone.svelte';

  import uploadEmoji from '../upload-emoji.js';

  const SET_ICON_URL = chrome.runtime.getURL('images/icon_128.png');

  let uploads = [];
  let uploadsStatusById = {};

  async function sleep (msec) {
    return new Promise(resolve => setTimeout(resolve, msec));
  }

  function changeUploadsStatus (fileId, type, message) {
    uploadsStatusById = {
      ...uploadsStatusById,
      [fileId]: { type, message },
    };
  }

  function changeUploads (file, fileId) {
    uploads = [...uploads, {
      file,
      id: fileId,
    }];
  }

  async function uploadFiles (files) {
    for (const file of files) {
      const id = uuid.v4();

      try {
        changeUploadsStatus(id, 'uploading', 'Uploading...');
        changeUploads(file, id);
        await uploadEmoji(file, id);

        changeUploadsStatus(id, 'success', 'Successfully Uploaded.');
      } catch (err) {
        changeUploadsStatus(id, 'error', err.message || err);
      }

      await sleep(5000);
    }
  }

  async function handleFilesAdded (event) {
    const files = event.detail;

    await uploadFiles(files);
  }
</script>

<style>
  .neutral-face-emoji-tools {
    border: var(--color-slack-border) 1px solid;
    border-left: var(--color-neutral-face-emoji-tools) 3px solid;
    margin: 0 0 25px 0;
    padding: 25px;
    background: white;
  }

  .icon.heading {
    margin: 0 5px 0 0;
    height: 1.25em;
    vertical-align: -25%;
  }

  .input-note {
    font-size: .9rem;
    line-height: 1.25rem;
    color: var(--color-text-gray);
  }

  .uploads {
    list-style-type: none;
    margin: 0;
    font-size: 0.9rem;
  }
</style>

<div class="neutral-face-emoji-tools">
  <h4 class="heading">
    <img class="icon heading" src="{SET_ICON_URL}" alt="" />
    <span class="text">Bulk Emoji Uploader</span>
  </h4>
  <p class="subheading">Drag and drop images into the area below. Any images dropped there will be automatically uploaded using their filename as the emoji name.</p>
  <p class="input-note">Example: <span class="normal">"ditto.gif" will be added as "ditto"</span></p>
  <FileDropzone on:filesadded={handleFilesAdded} />
  <ul class="uploads">
    {#each uploads as upload (upload.id)}
      <Upload upload={upload} status={uploadsStatusById[upload.id]} />
    {/each}
  </ul>
</div>
