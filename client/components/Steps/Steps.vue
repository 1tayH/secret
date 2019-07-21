<template>
  <div class="container">
    <h1 class="title">How does it work?</h1>
    <h2 class="subtitle">
      A few words about the process
    </h2>
    <div class="columns step">
      <div class="column">
        <Step number="1"
              image="writing.svg">
          Write down your note and click the "store note" button.
        </Step>
      </div>
      <div class="column">
        <Step number="2"
              image="encrypted.svg">
          A private encryption key is generated and your secret
          note is encrypted while in the browser,
          <b>before</b> hitting our servers endpoint.
        </Step>
      </div>
      <div class="column">
        <Step number="3"
              image="secure_server.svg">
          The encrypted note itself is sent to our servers securely
          and you recieve a shareable link containing
          the decryption key.
        </Step>
      </div>
      <div class="column">
        <Step number="4"
              image="grave.svg">
          Once your note is read it will be sent right to the
          note graveyard, completely un-recoverable!
        </Step>
      </div>
    </div>
  </div>
</template>

<script>
import Step from '@/components/Steps/Step'

export default {
  components: { Step }
}
</script>
