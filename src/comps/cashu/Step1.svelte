<script>
    import { randomBytes,bytesToHex } from "@noble/hashes/utils";
    export let secret=''
    const randomSecret = () => {
        secret =bytesToHex(randomBytes(12))

    }
</script>
<div class="flex gap-2">
    <input bind:value={secret} type="text" class="input input-primary text-error" placeholder="secret🤐">
    <button class="btn btn-primary" on:click={randomSecret}>
        generate 
    </button>
</div>